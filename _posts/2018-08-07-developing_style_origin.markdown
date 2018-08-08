---
layout: post
title:      "Developing Style : Origin "
date:       2018-08-08 00:21:12 +0000
permalink:  developing_style_origin
---


What interests me the most when solving a programming problem is how to most elegantly and efficiently engineer the solution. A brute force solution is never as satisfying as one which runs asymtotically faster or one which allows easy expansion through code reuse. Of course most of the time the level of creativity you can achieve in a solution depends on the problem itself. So, sometimes I take a break from more straightforward work in order to solve something more conceptual. I tend to lose track of time and take breaks far less often when doing this as well.  The greatest part about expanding your ability on this front is that the skills are very abstract and are therefore for the most part language-agnostic. This allows you to become a better engineer, not just a better Java or Ruby programmer.

So how to get better at these types of problems? Obviously there's practice, but there are topics of research as well. The study of [algorithms and their asymptotic runtimes](https://www.khanacademy.org/computing/computer-science/algorithms) is probably the most essential. Hand in hand with that are [data structures](https://www.geeksforgeeks.org/data-structures/). Along a different thought process you can find an assortment of  techniques, such as [functional programming](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-1-1f15e387e536), [protocol oriented programming](https://www.appcoda.com/pop-vs-oop/) (used mostly by the Swift community, but applies to several languages), along with associated [design patterns](https://en.wikipedia.org/wiki/Software_design_pattern) which are essential to study because they allow you to tap into solutions many intelligent people spent a long time figuring out. 

The problem with all of this is that it's a lot of material - it's hard to know where to start, and if you can ever learn enough to have a broad enough scope of knowlege which will allow you to actually apply the skills. I'm hoping the answer is yes, it's possible. I've been programming for almost a year now, and I was self-taught for the first 7 months. It's time for me to dig into some topics that I have never formally covered in any classes in order to expland my style to the next level. 

Over the next few posts I hope to maintain this theme and dive into some specific knowledge in this area. For the remainder of this post, I want to briefly talk about an idea I recently instantiated as a small side project. It was my own thought process, so it will be nowhere near as enlightening as any topics listed above. 

I saw a brief snippet the other day about how engineers are programming robots which are able to perform relatively complex tasks when working together as a unit. Instead of having complex logic in a single robot, they were programmed based of a set of simple rules to abide by and they learned based of their own experience as well as the experiences of other robots in the swarm. ([Bio-inspired programming](https://en.wikipedia.org/wiki/Bio-inspired_computing)). I thought this was a really cool idea, but with my current limited skillset and lack of hundreds of small programmable robots, I couldn't implement this. Along the same lines (well, kind of but not really), I thought instead my short side-project could be implementing physics instead of biological rules into a program. I decided that would be easy to test a simulation of the moon orbiting the earth. Instead of programming the moon moving around the earth at a given angular velocity with a specific radius, however, I wanted to program a Mass class which has the appropriate characteristics, a Gravitational Force class, and a Universe for these two objects to live in. Theoretically, all I would have to do is look up the moon's relative velocity to the earth and its distance, and I could get a working model (if I kept the earth stationary and had no other masses in my universe). 

So I started building, making some rough estimations of reality so I could get done the project in a reasonable amount of time. The basic structure was: 
* Make a mass object have 3 dimentional attributes acceleration, velocity, and location. 
* Have masses able to accept forces (which are 3D vector reprisentations). 
* Create classes to reprisent a position in 3D space and a 3D vector. Add arithmatic methods to allow custom computation. 
* Have a GravitationalForce class inherit from the Force class. This will accept 2 masses and output a force magnitude and unit vector direction based off of the gravitational force equation F = G(m1* m2) / (r^2)
* Have a universe class which manages all masses and distribues necessary forces, as well as updates all masses within it a specified time interval. Be aware that the gravitational forces change as masses change location, so this must be kept current.
* Each mass keeps track of its relative position, computed from an array of all forces it maintains. This is updated via the relationships 
* F = m * a
* a_new = Total_Force / mass
* v_new = a* t + v_old
* d_new = 0.5 a * t^2 + v_old * t + d_old
* Ensure your units are converted properly

Although I haven't refactored yet, I'll add some snippets below, and I'll put it on github once it's presentable. 

```
class Vector
    attr_accessor :x, :y, :z

    def initialize(x,y,z)
      @x = x
      @y = y
      @z = z
    end

    def magnitude
      Math.sqrt(x**2 + y**2 + z**2)
    end


    def unit_vector
      mag = self.magnitude
      Vector.new(@x / mag, @y / mag, @z / mag)
    end

    def *(num)
      nx = @x * num
      ny = @y * num
      nz = @z * num
      Vector.new(nx, ny, nz)
    end

    def /(num)
      nx = @x / num
      ny = @y / num
      nz = @z / num
      Vector.new(nx, ny, nz)
    end

    def +(vector)
      nx = @x + vector.x
      ny = @y + vector.y
      nz = @z + vector.z
      Vector.new(nx,ny,nz)
    end

    def copy
      Vector.new(self.x, self.y, self.z)
    end

    def to_s
      "x: #{self.x}, y: #{self.y}, z: #{self.z}"
    end

  end

```

And the part of the mass class which updated its relative positions:

```

  def update
    t = Time.now - last_update
    t *= 1000 # speed up time by a factor of 1000 so you can actually simulate the moon's orbit without waiting 27 days
    update_acceleration
    update_velocity(t)
    update_location(t)
    @last_update = Time.now
  end


  def combined_force_vector
    @forces.map{|force| force.force_vector_for(self)}.reduce{|cum, curr| cum + curr}
  end

  def combined_force_magnitude
    self.combined_force_vector.magnitude
  end

  def combined_force_direction
    self.combined_force_vector.unit_vector
  end

  def update_acceleration
    f_v = combined_force_vector
    @old_acceleration = @acceleration
    @acceleration = f_v / self.amt
  end

  def update_velocity(t)
    @old_velocity = @velocity
    @velocity = ((@acceleration + @old_acceleration) / 2 * t) + @velocity
  end

  def update_location(t)
    @location = ((@acceleration + @old_acceleration * 0.25) * (t ** 2)) + (@old_velocity * t) + @location
  end
```


I was able to get the following orbital data (I implemented both masses at the same place in the z-plane to keep everything 2D because I didn't have 3D graphing software):

![Moon Orbit](https://www.dropbox.com/s/am35n1nj87mu1j7/moon_orbit_final.png?dl=0)

What I was trying to get from this exercise besides practice was to see the advantages vs disadvantages of programming a set of rules (in this case physics) to accomplish a task rather than just programming out the task specifically. It probably would have been easier and taken less time to program a moon orbital directly (ie specify the radius of the orbital, and the angular velocity, and that's pretty much it). Also, another positive for programming an orbit is that it would allow you to create orbits that may not necessarily work under the laws of physics - ie you could have an orbit which is too slow for how close the masses are or too fast for how far apart they are but still just make the orbit work. Using the laws of physics you would be worried about collision in the former and an escape in the latter. However, there are advantages to programming a set of rules for your environment. In the version using the laws of physics, the code allows you to expand much more easily, creating more complex workings with only a little extra effort. Just by adding 5-10 lines of code to model elastic and inelastic collisions, I could simulate a ball bouncing on the earth's surface. If I had just coded an orbital that would require a fresh start to simulate. With the physics rules model, you can create a spaceship which can slingshot off the moon towards another mass, or a mass orbiting another mass which itself is orbiting a third mass, all of which have speed in all dimensions. 

So I guess my takeaway is that if you can figure out a set of rules which do not limit the outcome you want your program to have, you can use those rules to create an environment which governs itself and does not need to be told what to do by you as much, and that can reduce how much effort you need to put in as your code grows more complex. The challenge now is to generalize this lesson and be able to apply the concept to problems encountered in the future. 

Next time I'll get into some topics discussed above, which I think will be a lot more worthwhile to investigate than a fake moon orbital. 

Only if you're interested - I found a bit of interesting behavior this project with regards to the behavior of the earth in relation to the initial conditions of the system. I started the earth stationary at (0,0) and the moon  travelling in the +y direction and displaced from the earth in the +x direction. With that model (specifically, with the earth stationary at the beginning), the earth was oscillitory in nature in the x direction, with one period being a full moon orbital. However, it had a constant positive displacement in the y direction for each orbit of the moon, meaning the earth was constantly moving forward in the y direction instead of oscillating back and forth (as it was in the x direction). To counter this I had to give the earth a slight initial velocity in the -y direction, which makes the simulation more realistic. It may be an interesting exercise to think about why this happened, and why it is more true to reality to start the Earth off with a velocity in the -y direction. I did not think of this before assigning initial conditions, but it was fun figuring out why this happened after I identified the behavior.


