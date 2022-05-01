# Gary's Sauce

The zip file seems to contain some FPGA prorgramming language, verilog. It describes the design of a computer that has an input (the flag) represnted by "SW". You can prove this just by looking at the `flag_tb.v` file. The zip also contained 5 images that show 4 of the 8 switches (SW) and several other components of a simluation. After closely examining the source code files you will realize the SW (the bottom 4 components in the image) are actually part of the flag. You only see part of the flag though. We see 4 switches, and there are 8 switches  in byte so this means we know 4/8 bits for each character of the entire flag.  Bits 0,3,5,6 and the ones we have. We know that ascii ends at 127, so bit 7 must be zero. We still need to find bits 1, 2 and 4. We also see a few other components in the computer. A mask of 0xff, and a display of a byte in hex represented by `disp` in the images. Examining the code we can see that it multiplies the bottom 4 bits of the flag with  the top 4 bits of the flag. It then it XOR's it with the constant 0xd  (this code is in  `garys_sauce.v` and `myxor.v`). This was probably the worst part for me. It took me a while to figure out that 8'hd which means 0xd hex. The result of this is then displayed on the `disp`.

So if we XOR all the numbers in the `disp` by 0xd then we know what the product of the  flag bits are. The product is the result of the bottom 4 bits and top 4 bits of each character.  You can see this multiply happen in the source in `garys_sauce.v`. We can also see that there are 38 total characters by looking at the image files.

Using this information, we need to narrow down the possible results. The fastest way to arrive at the answer was to just try every number that a valid ascii character could be. This means only allowing characters that match the 4 bits we know about. Then we  multiply do the 4 bit multiply, do the 0xd XOR, then compare with the expected result (`disp`). By limiting the possible ascii characters to just hex numbers, brackets and letters you can narrow down the possible character for each spot to exactly 1 character. And it turns out the flag is actually reversed, which was really easy to figure out once you saw it You might have thought the first character was "f"? It's reveresred so its `}`.

The hard part about this challenge was writing the 4 bit states of all 38 characters EXACTLY into a script to achieve this, 1 bit wrong and you got something that resembles a flag but isn't the answer. As well as the 38 disp outputs. This was also kind of tricky because the "white" lines in the image did not line up with the clock cycle so I had to use a ruler to write down every bit one by one (basically every "disp" hex character you see where the BTNL switch is on, the 4 SW bits are the values during that moment.). Once that was done it was relatively easy to get the result

The final script looks like this:

garys\_source.rb


    sw0 = []
    sw3 = []
    sw5 = []
    sw6 = []
    
    #p1
    bits = [
    [1,1,1,1],
    [0,1,0,0],
    [1,1,0,1],
    [1,1,0,1],
    [1,1,0,0],
    [0,1,0,0],
    [0,1,0,0],
    [1,1,0,1],
    [0,1,0,0],
    ]
    d = [0x56,0xb,0xb,0x13,0x15,0xb,0x1,0x1f,0xd]
    bits.each do |set|
      sw6.push set[0]
      sw5.push set[1]
      sw3.push set[2]
      sw0.push set[3]
    
    
    
    end
    
    
    
    #p2
    bits = [
    [1,1,0,1],
    [0,1,0,0],
    [0,1,0,0],
    [0,1,1,0],
    [0,1,1,1],
    [0,1,0,0],
    [1,1,0,0],
    [0,1,0,0],
    [1,1,0,1],
    ]
    d += [0x13,0xd,0xb,0x15,0x16,0xb,0x29,0xd,0x1f]
    bits.each do |set|
      sw6.push set[0]
      sw5.push set[1]
      sw3.push set[2]
      sw0.push set[3]
    end
    
    
    
    
    
    #p3
    bits = [
    [0,1,0,1],
    [1,1,0,1],
    [0,1,0,0],
    [0,1,0,0],
    [1,1,0,0],
    [0,1,0,1],
    [0,1,0,0],
    [0,1,0,0],
    [0,1,1,0],
    ]
    d += [0x2,0x1f,0xb,0x1f,0x29,0x18,0xd,0x1,0x15]
    bits.each do |set|
      sw6.push set[0]
      sw5.push set[1]
      sw3.push set[2]
      sw0.push set[3]
    end
    
    
    
    
    
    #p4
    bits = [
    [0,1,0,0],
    [1,1,0,0],
    [0,1,0,0],
    [0,1,1,0],
    [0,1,0,0],
    [0,1,0,0],
    [1,1,1,1],
    [1,1,0,1],
    [1,1,0,1],
    ]
    d += [0x1f,0x15,0xb,0x15,0xd,0x1f,0x40,0x27,0xb]
    bits.each do |set|
      sw6.push set[0]
      sw5.push set[1]
      sw3.push set[2]
      sw0.push set[3]
    end
    
    
    
    
    
    
    #p5
    bits = [
    [1,1,1,0],
    [1,1,0,0],
    ]
    d += [0x45,0x29]
    bits.each do |set|
      sw6.push set[0]
      sw5.push set[1]
      sw3.push set[2]
      sw0.push set[3]
    end
    
    
    
    
    chars = []
    d.count.times do |idx|
      disp = d[idx]
    
      found = false
      256.times do |n|
        next if n < 48 
        next if n > 128
        p1 = n&0xf
        p2 = (n&0xf0)>>4;
        product = p1*p2
        product = product & 0xff
        product ^= 0xd
        if (n&1)==sw0[idx] and ((n&8)>>3)==sw3[idx] and ((n&32)>>5)==sw5[idx] and ((n&64)>>6)==sw6[idx]
          if product == disp
            chars.push n.chr
            found = true
          end
        end
      end
    end
    puts chars.join("").reverse
    
    
Run that in ruby and out comes the flag

  flag{6082d68407f62c5c0f29820e0c42dea2}
