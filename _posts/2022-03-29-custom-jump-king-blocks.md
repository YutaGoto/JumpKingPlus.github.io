---
layout: blog
title: Adding Custom Blocks to Jump King
subtitle: Behind the scenes of the first block update
date: 2022-03-26 16:45:35 +0100
Author: Elisiah
tags: [blog]
comments: false
toc: false
pinned: false
hidden: false
coverImage: https://raw.githubusercontent.com/JumpKingPlus/JumpKingPlus.github.io/www/images/Banner170.png
---

<!-- more -->

<img class="thumbnail" src="{{page.coverImage}}" alt="{{page.title}}" title="{{page.title}}" />

## Introduction

Most of Jump King's platforming takes place on solid blocks of ground and so I sought to create custom blocks that Modders could use to spice up the gameplay in their custom maps. These mods are packaged in our project [JumpKingPlus]({{ site.url }}).

It is pivotal to first understand how Jump King creates its levels and blocks.
A level screen is a 60x45 image in which every pixel constitutes an 8x8 square of pixels in the game (resulting in a 480x360 game screen).

Each pixel in the image determines the type of block in game by its colour.
For example a sand block has the colour: `Red: 255, Blue: 106, Green: 0`, and so when a pixel of that exact colour code is found the game assigns that region of the screen the sand properties and hitboxes.

<br>

## Creating the Blocks

I will cover in vague detail the steps to create any blocks and then give two examples of blocks I have made.

The first step I took towards creating a block was seeing how the in game ones function. I determined they were defined in the following way:
- A `BlockName.cs` file which defines local properties of how they function
- A `BlockName enum` file, optionally, if the block has multiple types
- The block is defined with its colour code in `LevelManager.cs` with a collision variable
- The block is assigned the tag SOLID, NON SOLID or in some rare instaces both - in `LevelScene.cs` with a collision variable
- The block has a collision function in `BodyComp.cs` which defines how the player will interact with hitting the blocks walls (y+, y-, x+, x-) 

My example blocks to show the steps I took are as follows:
- A **Portal** that would move the player to a set of coordinates on the same screen
- **One Way** blocks that can only be passed through from one side (useful for puzzles)

### The Portals

To create portals I had to iften consider and work around the limitations we had.

Initially I had planned to have two portals that link to eachother, however we only have 3 ways of gathering information (RGB) and if we linked the portals using a given value, lets say R, we would also have to match a value to show they are both portals to differntiate them from other block types, lets say B, at which point we would only have one piece of information left with 255 possibilities which isnt enough space to hold coordinate data.

To work around this I assigned Blue as the "Portal" colour, such that it could be differentiated from other blocks. This allowed me the full `0-255` range for both other input sources (RG); given the blocks location data is simply its pixel coordinate in the image all the data needed is the location the user would come back out. hence, **R** and **G** are the **X** and **Y** coordinates of exit, respectively.

![RGB Info](https://elisiah.github.io/assets/rgb%20information.png)

Since the portals now hold **in-location** and **out-location** data we can simply "pseudo link" portals by just having the exit location be next to the other drawn portal.

The collision data for this block is very simple, it is non-solid and simply alters the player coordinates on passing through any of the walls of the block.

<iframe width="100%" height="480" src="https://www.youtube.com/embed/gSpWIzu0uZE" frameborder="0" allowfullscreen></iframe>

### One Way Blocks

Given I wanted one way blocks for every interactable direction I created the `enum` file referenced earlier.

In terms of the colour code for this block the criteria was:
- Take up minimal space so more blocks in the future wont have ID clashes
- Contain the information of which enum type the block is

The first criteria was simple, I just assigned B and G to both be the same value (I chose 65).

The second criteria wasn't too difficult either, I simply set the default to `65` to match G and B above; and then also assigned the codes `R = 66, 67, 68` for the further 3 types of one way block.<br> Hence, the `enum = Colour.R - 65`.

The collision function for this block would check the wall the player collided with and make sure it matched the enumerable type of the block hit. If the check succeeded the player speed would be stopped and a collision would occur; alternatively, nothing had to happen as the player was intendeed to just pass through.

<iframe width="100%" height="480" src="https://www.youtube.com/embed/WZf6RXXgBks" frameborder="0" allowfullscreen></iframe>