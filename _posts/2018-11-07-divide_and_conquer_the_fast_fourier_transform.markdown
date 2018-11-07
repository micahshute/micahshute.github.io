---
layout: post
title:      "Divide and Conquer the Fast Fourier Transform"
date:       2018-11-07 01:55:31 -0500
permalink:  divide_and_conquer_the_fast_fourier_transform
---


## What is the Fourier Transform and why should I care

Simply put, the Fourier Transform allows humans or machines to see time domain signals in the frequency domain. Common applications are data visualization in oscilloscopes and function generators, radar, sonar, [spectral analysis](https://en.wikipedia.org/wiki/Fourier-transform_spectroscopy), digital communication signal processing, and instrument monitoring.


There is also a broad range of uses for the Fourier Transform beyond visualization or even an end game of frequency analysis. Linear Time Invariant (LTI) systems and signals respond to each other through [convolution](https://en.wikipedia.org/wiki/Convolution), which can be a mathematically impractical operation to evaluate. These system outputs can be modeled accurately by simply multiplying their Fourier Transforms together and then converting them back to the time domain, given that we avoid aliasing when doing so. 

Lastly, a lot of data compression uses algorithms similar to the Fast Fourier Transform. For example, [JPEGs use the Discrete Cosine Transform](https://en.wikipedia.org/wiki/Discrete_cosine_transform) which is very similar to the Discrete Fourier Transform.

## Why and how does it work

This is a question with a long answer. Here's the short and dirty:

This is the equation to convert a time domain signal x(t) to a frequency domain signal X(f):  

 <a href="http://www.codecogs.com/eqnedit.php?latex=$X(f)&space;=&space;\int_{-\infty}^{\infty}{x(t)}e^{-j2\pi&space;t}dt$" target="_blank"><img src="http://latex.codecogs.com/gif.latex?$X(f)&space;=&space;\int_{-\infty}^{\infty}{x(t)}e^{-j2\pi&space;t}dt$" title="$X(f) = \int_{-\infty}^{\infty}{x(t)}e^{-j2\pi t}dt$" /></a>


Where t is time and f is frequency in Hz.


If you haven't seen this before it doesn't make much sense. Without getting caught up in the math, here's what you need to know:
 1. Almost all signals that exist can be made up of a summation of cosine and sine signals, even aperiodic continuous time signals. 
    - For example, a decaying exponential signal 

<a href="http://www.codecogs.com/eqnedit.php?latex=$x(t)&space;=&space;e^{-t}u(t)$" target="_blank"><img src="http://latex.codecogs.com/gif.latex?$x(t)&space;=&space;e^{-t}u(t)$" title="$x(t) = e^{-t}u(t)$" /></a> has the frequencies: <a href="http://www.codecogs.com/eqnedit.php?latex=X(f)&space;=&space;\frac{1}{1&space;&plus;&space;j2\pi&space;f}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X(f)&space;=&space;\frac{1}{1&space;&plus;&space;j2\pi&space;f}" title="X(f) = \frac{1}{1 + j2\pi f}" /></a>

whose magnitudes look like this:
    - ![Fourier Transform](https://thepracticaldev.s3.amazonaws.com/i/qmsde0ac0zkp70meiz62.png)
    - You may recognize that a capacitor responds to a voltage impulse via an exponential decay, slowly discharging its built up voltage over time. So, the fact that the frequency makeup of the exponential decay is large for lower frequencies and small for higher frequencies makes sense intuitively, because the voltage across a capacitor will only look like the input when the input changes values slowly enough to allow this discharge process to take place. This is why the frequency range looks the way it does in this case.
    - Note: Negative frequencies are part of both sine and cosine signals. it is what keeps time signals "real" when we mathematically model them as will be explained below. 
   

 2. We can make mathematically convenient equations using Euler's property that:
    - <a href="http://www.codecogs.com/eqnedit.php?latex=e^{j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;&plus;&space;j\sin(2\pi&space;ft)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;&plus;&space;j\sin(2\pi&space;ft)" title="e^{j2\pi ft} = \cos(2\pi ft) + j\sin(2\pi ft)" /></a>
    - <a href="http://www.codecogs.com/eqnedit.php?latex=e^{-j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;-&space;j\sin(2\pi&space;ft)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{-j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;-&space;j\sin(2\pi&space;ft)" title="e^{-j2\pi ft} = \cos(2\pi ft) - j\sin(2\pi ft)" /></a>
    - <a href="http://www.codecogs.com/eqnedit.php?latex=\cos(2\pi&space;ft)&space;=&space;\frac12(e^{j2\pi&space;t}&space;&plus;&space;e^{-j2\pi&space;t})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\cos(2\pi&space;ft)&space;=&space;\frac12(e^{j2\pi&space;t}&space;&plus;&space;e^{-j2\pi&space;t})" title="\cos(2\pi ft) = \frac12(e^{j2\pi t} + e^{-j2\pi t})" /></a>
    - <a href="http://www.codecogs.com/eqnedit.php?latex=\sin(2\pi&space;ft)&space;=&space;\frac{1}{2j}(e^{j2\pi&space;t}&space;-&space;e^{-j2\pi&space;t})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sin(2\pi&space;ft)&space;=&space;\frac{1}{2j}(e^{j2\pi&space;t}&space;-&space;e^{-j2\pi&space;t})" title="\sin(2\pi ft) = \frac{1}{2j}(e^{j2\pi t} - e^{-j2\pi t})" /></a>
        - Don't be confused about the imaginary numbers. They are a way to show phase angle data in the frequency domain. Real time signals will never become "imaginary" time signals - the imaginary parts will always cancel out to give you a mixture of sines and cosines. 
        - Note that this shows why negative frequencies are necessary to model sinusoids in this way.

So, if we know from (1) that all signals can be made up of a summation of sines and cosines, we can manipulate that fact to retrieve those cosines and sines from a time signal. These retrieved sines and cosines are our frequency domain signal. It is done mathematically via the equation above, like was shown in the example.

## Ok ... so I don't see how I can make a program that does this

As of now, our equation acts on a continuous time (CT) signal via an infinite integral. That is not conducive to digital processing and programming. Before we can try to write a program to get the information we need, you have to do a few things to get a signal we can manipulate.

1. Turn the infinite CT signal into a discrete time, limited size signal. This can be done by sampling and quantization for a limited number of samples. If you sample at a rate >= twice the highest frequency component of the signal (ie the Nyquist frequency), you can avoid aliasing. This is usually done by dedicated hardware. However, we care now about how we use these samples, not how we get them.
2. Change the equation for getting the frequency domain of our signal so it can work with a size-limited array of sampled data points instead of an infinite continuous time signal. So, instead of an integral we need a summation, and instead of an infinite size we use the size of our sampled data array:
    - One more problem though - the frequency domain of an aperiodic signal, discrete or not, is still continuous - so we can't manipulate it on a computer. The answer to this is the Discrete Fourier Transform, a periodically sampled version of the Fourier Transform for a discrete time signal. To get this, we can replace the frequency term with a term that divides it into equal parts over the N samples: <a href="http://www.codecogs.com/eqnedit.php?latex=2\pi&space;f&space;\Rightarrow&space;\frac{2\pi&space;k}{N}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?2\pi&space;f&space;\Rightarrow&space;\frac{2\pi&space;k}{N}" title="2\pi f \Rightarrow \frac{2\pi k}{N}" /></a> where we are evaluating for the k^th sample and there are N total samples.

        - If you are curious if we can recreate the time domain samples accurately from this sampled version of the frequency domain, the short answer is yes, as long as we avoid time-domain aliasing by ensuring the total samples of our frequency domain are greater than or equal to the length of our time domain sample. We can increase the sample rate of our frequency domain without increasing the length of our sampled data by adding zeros to the end of the time domain signal. This will make the samples of our frequency domain be closer together.
    - Anyway, the end story is that we get this equation to get a frequency domain transform function from an array of sampled data: 
    - <a href="http://www.codecogs.com/eqnedit.php?latex=X(k)&space;=&space;\sum_{n&space;=&space;0}^{N-1}{x(n)e^{-j\frac{2\pi&space;kn}{N}}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X(k)&space;=&space;\sum_{n&space;=&space;0}^{N-1}{x(n)e^{-j\frac{2\pi&space;kn}{N}}}" title="X(k) = \sum_{n = 0}^{N-1}{x(n)e^{-j\frac{2\pi kn}{N}}}" /></a>

## How do you compute it

Now that we have the equation, this part is easy!

```ruby
signal = ->(t){Math.cos(2 * Math::PI * t)} # 1 Hz cosine signal
sample_times = (0..1000).map{ |int| int / 100.0} # Sample for 10 seconds every 0.01 seconds (100 times per period over 10 periods)
data = sample_times.map{ |time| signal.call(time) } # turn time into sample values

#Discrete Fourier Transform
ft = []
for k in 0...data.length do
    tot = 0
    data.each_with_index do |x_n, n|
        tot += x_n * Math::E ** (Complex(0,-1) * 2.0 * Math::PI * k * n / data.length.to_f)
    end
    ft << tot
end
```
Too easy! Done, right?

Not quite. You get the right answer, but that looks like a nested loop to me. &#1012;(n^2) is rearing its ugly head to let us know we can't do any real-time processing with this algorithm.

Since the FFT is needed in so many real-time applications, and sampled signals are usually very long, we need to find a way to severely cut down on processing time.  

## How to compute it well: Divide and Conquer

So here's where the Fast Fourier Transform (FFT) comes in, which allows us to use this signal in real time. 

First, let's get intuition to what the shortcuts are that we can take advantage of, which makes it advantageous and possible to cut this problem down into smaller sub-problems. The characteristics we want to use are twofold, and come from the fact that we are manipulating [complex roots of unity](https://en.wikipedia.org/wiki/Root_of_unity) by multiplying by 

<a href="http://www.codecogs.com/eqnedit.php?latex=e^{-j\frac{2\pi&space;kn}{N}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{-j\frac{2\pi&space;kn}{N}}" title="e^{-j\frac{2\pi kn}{N}}" /></a> 

on each iteration. What does that mean? Well, remember from above that 

<a href="http://www.codecogs.com/eqnedit.php?latex=e^{j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;&plus;&space;j\sin(2\pi&space;ft)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{j2\pi&space;ft}&space;=&space;\cos(2\pi&space;ft)&space;&plus;&space;j\sin(2\pi&space;ft)" title="e^{j2\pi ft} = \cos(2\pi ft) + j\sin(2\pi ft)" /></a> 

Let's first see what an example of that looks like; I did a little back-of-a-napkin demonstration:
![Real Imaginary](https://thepracticaldev.s3.amazonaws.com/i/dbgcpuajxw7mjio1p867.jpg)  



So you can think of the Real and Imaginary axis like East/West and North/South lines, and <a href="http://www.codecogs.com/eqnedit.php?latex=e^{j2\pi&space;ft}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{j2\pi&space;ft}" title="e^{j2\pi ft}" /></a> as a compass needle that points to the angle <a href="http://www.codecogs.com/eqnedit.php?latex=2\pi&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?2\pi&space;f" title="2\pi f" /></a> and changes with time t. The properties we want to take advantage of are based off of the situation where we divide the 2&#960; radians into N even segments, as we are doing to calculate the Discrete Fourier Transform. That can be visualized:  
  


![Complex Roots of Unity](https://thepracticaldev.s3.amazonaws.com/i/mpxvxaiqzda8evcw51kc.jpg)

Can you see the properties we want to exploit? They are:  

 1. The i^th partition is the complex conjugate of the N-i^th  partition. This means that if we have 8 partitions (and k = 1), the 1^st partition is <a href="http://www.codecogs.com/eqnedit.php?latex=e^{j2\pi&space;/&space;8}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{j2\pi&space;/&space;8}" title="e^{j2\pi / 8}" /></a> and the -1^st partition (ie the 7th partition) is <a href="http://www.codecogs.com/eqnedit.php?latex=e^{j14\pi&space;/&space;8}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{j14\pi&space;/&space;8}" title="e^{j14\pi / 8}" /></a>. 
    - n = 1:  <a href="http://www.codecogs.com/eqnedit.php?latex=\,\,e^{j2\pi&space;/8}&space;=&space;\cos(2\pi/8)&space;&plus;j\sin(2\pi/8)&space;=&space;0.707&space;&plus;j0.707" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\,\,e^{j2\pi&space;/8}&space;=&space;\cos(2\pi/8)&space;&plus;j\sin(2\pi/8)&space;=&space;0.707&space;&plus;j0.707" title="\,\,e^{j2\pi /8} = \cos(2\pi/8) +j\sin(2\pi/8) = 0.707 +j0.707" /></a>
    - n = 7:  <a href="http://www.codecogs.com/eqnedit.php?latex=\,\,e^{j14\pi/8}&space;=&space;cos(14\pi/8)&space;&plus;&space;j\sin(14\pi&space;/8)&space;=&space;0.707&space;-&space;j0.707" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\,\,e^{j14\pi/8}&space;=&space;cos(14\pi/8)&space;&plus;&space;j\sin(14\pi&space;/8)&space;=&space;0.707&space;-&space;j0.707" title="\,\,e^{j14\pi/8} = cos(14\pi/8) + j\sin(14\pi /8) = 0.707 - j0.707" /></a> 
    - As you can see, the 1 and N-1 signals are related because their real components are the same, and their imaginary components are negatives of each other. You can see that visually above.  

2_ This circle is periodic, so when the variable k begins to increase, we just loop around the circle as n increments.

**Now let's finally see how to exploit these characteristics using the Radix 2 FFT algorithm.**

1. We know that in order to divide and conquer, we want to break down a problem in halves (or some radix). We notice from our drawing that if we take the even partitions in the case of 8 partitions, we will be left with the partitions as if we had 4 complex roots of unity instead of 8 (ie 4 even divisions of 2&#960;), thus giving us a smaller problem to solve, which is what we want. Mathematically, let's see what it looks like if we broke the signal up into even and odd samples: 
   - ![Summation](https://thepracticaldev.s3.amazonaws.com/i/sgd2cw831uf0xa4pjrld.gif)
   - This reduces to an equation of 2 sub-arrays of data which are half the size of the original dataset, and will be multiplied by the roots of unity of N/2 instead of N, essentially creating 2 identical sub-problems:  
        - ![Subproblems](https://thepracticaldev.s3.amazonaws.com/i/jbww0povagr8qtvp1nov.gif)
    - We can now just recurse with our even and odd subarrays until you hit the base case of N = 1 (here n and k will be zero so you are just multiplying this one value by e^0 = 1), and return that value as the DFT of itself. Then combine per the equation above.
2. Awesome, but there is still one more thing we can do. Remember how we said that the i^th and N-i^th roots are complex conjugates of each other (ie, the roots of unity drawing is a mirror image across the x axis)? Well, that means that once we solve the two sub-problems for size of N/2, we can combine them to get the N DFT values by creating their corresponding "mirror image" values at the same time. This will stop us from having to loop through the N/2 sub-problems' data twice to create the N data values required to make X(k) one level up. We want to take the k^th result of the DFT of the even array and we add it to  <a href="http://www.codecogs.com/eqnedit.php?latex=e^{-j\frac{2\pi&space;k}{N}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{-j\frac{2\pi&space;k}{N}}" title="e^{-j\frac{2\pi k}{N}}" /></a> times the k^th result of the DFT of the odd array, and put it into an array that will be for the 0...N/2 values of X(k). We again take the k^th result of the DFT of the even array but now we subtract from it <a href="http://www.codecogs.com/eqnedit.php?latex=e^{-j\frac{2\pi&space;k}{N}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?e^{-j\frac{2\pi&space;k}{N}}" title="e^{-j\frac{2\pi k}{N}}" /></a> times the k^th result of the DFT of the odd array, and put it into an array that will be for the N/2...N values of X(k). We do this N/2 times. Lastly, we append the N/2...N array to the end of the 0...N/2 array to get the full answer for the next level up, X(k). In an equation, each level's combine step looks like this: 
  
   - <a href="http://www.codecogs.com/eqnedit.php?latex=X(k)&space;=&space;FFT(even\,&space;subarray)[k]&space;&plus;&space;e^{-j\frac{2\pi&space;k}{N}}FFT(odd\,subarray)[k]" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X(k)&space;=&space;FFT(even\,&space;subarray)[k]&space;&plus;&space;e^{-j\frac{2\pi&space;k}{N}}FFT(odd\,subarray)[k]" title="X(k) = FFT(even\, subarray)[k] + e^{-j\frac{2\pi k}{N}}FFT(odd\,subarray)[k]" /></a> for <a href="http://www.codecogs.com/eqnedit.php?latex=0\leqslant&space;k&space;<&space;N/2" target="_blank"><img src="http://latex.codecogs.com/gif.latex?0\leqslant&space;k&space;<&space;N/2" title="0\leqslant k < N/2" /></a>

   - <a href="http://www.codecogs.com/eqnedit.php?latex=X(k)&space;=&space;FFT(even\,&space;subarray)[k]&space;-&space;e^{-j\frac{2\pi&space;k}{N}}FFT(odd\,subarray)[k]" target="_blank"><img src="http://latex.codecogs.com/gif.latex?X(k)&space;=&space;FFT(even\,&space;subarray)[k]&space;-&space;e^{-j\frac{2\pi&space;k}{N}}FFT(odd\,subarray)[k]" title="X(k) = FFT(even\, subarray)[k] - e^{-j\frac{2\pi k}{N}}FFT(odd\,subarray)[k]" /></a> for  <a href="http://www.codecogs.com/eqnedit.php?latex=N/2\leqslant&space;k&space;<&space;N" target="_blank"><img src="http://latex.codecogs.com/gif.latex?N/2\leqslant&space;k&space;<&space;N" title="N/2\leqslant k < N" /></a>

3_ A necessary detail:  

   - This premise of being able to divide the problem by 2 and make even partitions of the unit circle requires that the sample data array be a power of 2. This is a simple metric to meet. All you have to do is add 0s to the end of your data until your data array's length is a power of 2. This ensures the algorithm works, and it creates closer "sample points" in your Discrete Fourier Transform, making the visualization better.

## Show me how you implemented it

```ruby
class Radix2Strategy 

    E = Math::E
    PI = Math::PI

    attr_reader :data

    def initialize(data: )
        @data = data
        zero_fill
    end

    def calculate(data = @data)
        recursive_fft(data)
    end

    private

    def recursive_fft(arr)
        n = arr.length
        return arr if n == 1
        w_n = E ** ((-2 * PI * Complex(0,1)) / n)
        w = 1
        a0 = get_even(arr)
        a1 = get_odd(arr)
        y0 = recursive_fft(a0)
        y1 = recursive_fft(a1)
        y = Array.new(n, 0)
        for k in 0...(n/2) do
            y[k] = y0[k] + w * y1[k]
            y[k + n/2] = y0[k] - w * y1[k]
            w = w * w_n
        end
        return y
    end

    def get_even(arr)
        even = []
        arr.each_with_index { |a, i| even << a.dup if i.even? }
        even
    end

    def get_odd(arr)
        odd = []
        arr.each_with_index { |a, i| odd << a.dup if i.odd? }
        odd
    end

    def zero_fill
        len = self.data.length
        @data = @data.concat Array.new(closest_pow_of_2(len) - len, 0)
    end

    def closest_pow_of_2(num)
        2 ** Math.log(num,2).ceil
    end
end
```

