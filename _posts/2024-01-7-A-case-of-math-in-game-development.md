---
layout: post
title: A case of math in game development
---

# Background
In the past, I've always heard people mention "Oh you’re studying Computer Science? You’ll really need math!" and I never understood it. To me, programming was about building things, it wasn’t about applying random formulas or solving for some value of x. I always remarked: “Uh… yeah sure.” without really meaning it. I thought about programming related concepts like web design and could not understand what they had to do with math. However, over the years I’ve been exposed more to different parts of programming and have seen where mathematics comes into play. Game development is the most recent example I’ve come across, so I’ve decided to write about it.

A little background, over the past year I’ve been developing a game using a game engine called Godot. Godot is a free open source game engine that allows one to build 2D and 3D games. The game I’m working on is a simple 2D game without any crazy physics simulations or anything that would make one think that it would require complex math. However, over the course of the game’s development, situations have arisen where mathematics has become useful to know.

# Introduction
One of the most striking examples of a mathematical topic useful in game development are vectors. A useful way to think of vectors are as arrows in space, with a magnitude (length) and a direction. Vectors can be represented as lists of numbers, similar to fixed-size arrays in many programming languages. If you haven’t been exposed to vectors before, I highly recommend viewing this [video](https://www.youtube.com/watch?v=wXI9_olSrqo) to get a quick grasp of the concept.

My game’s gameplay is very focused on projectiles. Enemies, bosses, and the player can fire projectiles. Projectiles are the main mechanism the player uses to interact with the world.  When a projectile is being fired, be it from an enemy, a boss or the player, there are two main things that determine the movement of the projectile, the speed and the direction. The direction is the most important part since there are infinitely many directions that a projectile could be fired from. The speed is also important, since it can’t be too large (or else the projectiles will be impossible to avoid; meaning the game will be too difficult) and it can’t be too small (or else the game will be too easy). However, generally the speed is constant and doesn’t have to be computed.

# Vectors
Both of these problems can be addressed with vector operations. There are many special operations that can be performed on vectors, which could be addition, subtraction or some other special operation. If one needs to compute the direction that the projectile needs to travel to go from point B to A, one just needs to subtract the two vectors like so: A - B.

Note the order, A - B gives you the vector that goes from B to A, meanwhile B - A gives you the vector that goes from A to B (which goes in the opposite direction). 

If A is a vector representing the position of the player and B is a vector representing the position of an enemy, we can compute the direction of a projectile fired by an enemy towards the player. See the following example code,

```gdscript
func fire_enemy_projectile(enemy, player):
    var pos_a = player.position
    var pos_b = enemy.position


    var direction = pos_a - pos_b


    # Summon the projectile at the enemy’s position (B), 
    # heading in the calculated direction (A - B).
    summon_projectile(pos_b, direction)
```
# Problem
However this causes another problem, the magnitude of the vector that results from the calculation of A - B, is bigger when it’s farther way from the player and smaller when the enemy is closer to the player. This behavior could be useful for some other use case, but right now, it’s better for the projectile’s speed to be the same regardless of the distance between the targets. Vector math comes to save us again, as there is an operation called normalization that can be applied to vectors.

**Figure 1**: The direction vector when A (the player) is farther away from B (the enemy).

![figure 1]({{ site.baseurl }}/images/blogs/post_01/figure_01.png)

**Figure 2**: The direction vector when A (the player) is closer to B (the enemy).

![figure 2]({{ site.baseurl }}/images/blogs/post_01/figure_02.png)

The normalization operation can be thought of as a function that takes any vector, and returns a new vector heading in the same direction, but with the magnitude set to 1. If the direction vector wasn’t normalized, if the player was farther away from the enemy, the projectile would move faster, and slower if the player was closer. This would be confusing for people playing the game to say the least. 

**Figure 3**: The normalized direction vector when A (the player) is close to B (the enemy).

![figure 3]({{ site.baseurl }}/images/blogs/post_01/figure_03.png)

**Figure 4**: The normalized direction vector when A (the player) is farther away from B (the enemy).

![figure 4]({{ site.baseurl }}/images/blogs/post_01/figure_04.png)

See the following example code,

```gdscript
func fire_enemy_projectile(enemy, player):
    var pos_a = player.position
    var pos_b = enemy.position


    # Sets the calculated vector’s magnitude to 1, while 
    # the direction.
    var direction = normalize(pos_a - pos_b)


    # Summon the projectile at the enemy’s position (B), 
    # heading in the calculated direction (A - B).
    summon_projectile(pos_b, direction)
```

# What about the speed?
The code we have now successfully computes the direction that the summoned projectile should head in, but what about the speed? Currently there is no way to change the speed of a fired projectile. 

# Scaling
A vector can be multiplied by a number (scalar) using an operation called scaling. If we store our scalar in a variable, we should be able to multiply it with the direction vector to get a velocity vector. This way, one can use the normalized direction vector along with a desired scalar to compute the velocity of an enemy’s projectile. For example, if a vector is multiplied by a scalar with a value of 2.0, the resulting vector would have twice the original vector’s magnitude. 

**Figure 5**: The normalized direction vector after the enemy fires a projectile.

![figure 5]({{ site.baseurl }}/images/blogs/post_01/figure_05.png)

**Figure 6**: The velocity vector when A (the player) is close to B (the enemy). The scaling factor used in this figure is 1.5, hence, the magnitude (length) of the velocity is 1.5.

![figure 6]({{ site.baseurl }}/images/blogs/post_01/figure_06.png)

Armed with this insight, we can write our final code

```gdscript
func fire_enemy_projectile(enemy, player):
    var pos_a = player.position
    var pos_b = enemy.position


    var speed = 5.0


    # Sets the calculated vector’s magnitude to 1, while 
    # the direction.
    var direction = normalize(pos_a - pos_b)
    
    # Computes the velocity vector.
    var velocity = direction * speed


    # Summon the projectile at the enemy’s position (B), 
    # heading in the calculated direction (A - B).
    summon_projectile(pos_a, velocity)
```

# Closing remarks
This marks the end of this blog post, I hope it was interesting. I think math is an important tool in many fields, including in programming, and should not be something to be avoided. I encourage everyone to at least learn the math they use, be it in engineering, physics, chemistry, programming, etc. instead of just blindly applying math formulas. Knowing the “why” is sometimes more important than the application, as it can help one discern the difference between when you think you need a certain formula and when you actually need it.

