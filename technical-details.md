Technical Details
=================

There are three tricks used in the rates.gba demo:

1. 32K sample rate
2. 9-Bit samples
3. Dithering

32K Sample Rate
---------------

Achieving 32K sample rate on the GBA is quite difficult.

The normal technique for generating sound on the GBA is to pick a buffer size, and then update the
buffer every frame.  Unfortunately, this doesn't work for the 32K sample rate, due to the timing of
the LCD.

Instead, the technique I use is to use a timer for 311296 cycles, which fires an interrupt.  When
the interrupt fires, I swap the DMA sources, and set a flag indicating that the buffer needs to be
updated.

Inside the game loop, I check the flag every frame.  About 90% of the frames will have the flag set.
If it's set, I fill the buffer with the next chunk of music.

The most difficult part is ensuring that the VBlank interrupt doesn't interfere with the Timer
interrupt.

In order to ensure they won't conflict, I setup the Timer interrupt to be in sync with the VBlank
interrupt -- if they fire at the exact same moment, then they will stay in sync indefinitely.

In order to get them to fire at the exact same moment, I wait for VBlank to fire, wait a little bit,
then start the Timer.

Specifically, I wait 250475 cycles inside the IRQ before starting the Timer.  You can see that in
the code here:

```
@irqStartTimer:
  .begin
    // delay a certain number of cycles to align timer1 with vblank
    // this can require tuning, depending on a lot of factors, and
    // was a pain to get exactly right... the indicator will blink
    // when timer1 and vblank are exactly in alignment
    push  {lr}
    ldr   r0, =0x3d27b - 16
    bl    @cycleDelay

    // set IRQ handler
    ldr   r0, =0x03007ffc
    ldr   r1, [#@@irqHopAddr]
    str   r1, [r0]

    // start timer
    ldr   r0, =$REG_TM1CNT
    ldr   r1, =0xc1
    strh  r1, [r0]

    pop   {lr}
    b     @irqNone
@@irqHopAddr:
    .i32 @irqHop
    .pool
  .end
```

This value was difficult to figure out, and took a lot of testing.

This exact same technique is used for 16K and 65K, only the buffer size and timer driving the sample
rate changes.  So for 16K, the buffer is half the size, and the timer is twice as slow, and for 65K
the buffer is twice the size, and the timer is twice as fast.  Note that the sample rate timer is
different than the timer described above -- the timer described above doesn't drive samples, it
determines when to swap out the buffers.

9-Bit Audio
-----------

Generating 9-Bit audio also took some experimentation.  In the end, it's not practical for games to
use 9-Bit audio, but it's interesting to see it work for the purposes of a demo.

I setup DMA1/FIFO A at 100% volume, and put the top 8 bits in this buffer.

Then, I setup DMA2/FIFO B at 50% volume, and put the bottom bit in this buffer (as `00` or `01`).

Using 50% volume allows the final sample to access the lower bits.

I tested this on hardware to verify that you can indeed hear FIFO B at 50% with only `00` or `01` in
the buffer.  You can perform the same test by silencing FIFO A in the rates demo, here:

```
// uncomment this line to verify that 9-Bit audio will be audible
// it will silence the high 8-Bits, so you can hear the low 1-Bit
//.def $VERIFY_9BIT = 1
```

Dithering
---------

Dithering is achieved by storing the sine wave as 16-Bit.

When converting 16-Bit to 8 or 9-Bit, the discarded bits are used for dithering.

A random byte is generated for every sample.  If the discarded bits are less than the random number,
then the final value is rounded down.  Otherwise, it's rounded up.

You can see the 8-Bit dithering here:

```
      // read 16-bit sample
      mov   r7, r4, lsl #1
      ldrsh r7, [r6, r7]

      // split into sample + fractional part
      mov   r8, r7, asr #8
      and   r9, r7, #0xff

      // compare fractional part to random number
      cmp   r9, r10
      blt   @@ditherDone
      // round up sample (but don't exceed 0x7f)
      cmp   r8, #0x7f
      bge   @@ditherDone
      add   r8, #1
@@ditherDone:
      mov   r7, #0

      strb  r8, [r0] // FIFO A
      strb  r7, [r1] // FIFO B
```
