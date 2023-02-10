# Portfolio

Welcome to my portfolio! I love working with Unity and C#! Originally I started off as an FOSS developer, working with Linux and open source tools. However after moving to Unity, I fell in love with C#. The language is really awesome, and gives the benefit of strongly typed languages with a lot of ease of convience features.

I started my first Unity project in 2018, and still remember jumping into the editor and being flooded with many cool new features. The journey was long, but I have consistently worked in the Unity editor ever since, and hope to share here my portfolio and some details of the process.

# RPG Game

I wanted to make a vertical slice for an RPG game. Wtih a vertical slice I could experiment with systems, while efficiently moving forward. I've on RPG games before and this time I wanted to put to use the lessons I've learned from the past. 
# Player Start

I wanted the player to start in a small closed space, as many old FPS games have done. I also wanted the player to start in a comfortable space, so they can gently ease into the game. To make the player more comforable I worked on a fire torch, including the comforting sounds of fire.

# Developing an AI Character

In this section we will go over the process of developing an AI character.

## Scene set up

We start off by creating a new empty scene. In this scene we will include our core functionality, such as the player character, the game manager, and other systems. (Note that we would like some of these to be prefabs but since we are still continually reworking these components we will turn them into prefabs once they are closer to completion.)


![Scene Setup](https://github.com/pjkw/Portfolio/blob/main/images/Scene%20Setup.png)

To start off we create a plane with a checkered material to keep things simple and focus just on our new character.

![Empty Scene](https://github.com/pjkw/Portfolio/blob/main/images/Empty%20Scene.png)

## Asset import

For the dragon, we will be using the Unka dragon asset from the Unity asset store: https://assetstore.unity.com/packages/3d/characters/creatures/unka-the-dragon-84283

Even with turning the skybox back on, the dragon is too dark in the shadows and we are losing details.

![Dragon](https://github.com/pjkw/Portfolio/blob/main/images/Dragon.png)

## Lighting

By default, Unity comes with the ambient light set to a middle gray, which can be a little flat. We will turn off ambient light by setting the color values to dark, effectively removing ambient light, and letting us set up our own lighting.

![Ambient Light](https://github.com/pjkw/Portfolio/blob/main/images/Ambient%20Light.png)

Now we will add three point lighting to the scene to light up the details in the shadows, and from different sides. Essentially we will have a main light, a rim light, and a key light.

![3 Point Lighting](https://github.com/pjkw/Portfolio/blob/main/images/Dragon%202.png)

After adding three point lighting, we are now getting much more details in the texture. However, from far away the details can blend since the dragon is black. In a dark area, such as a dungeon, we can add a moving light on top of the dragon to get extra detail, while still having our scene be dark. I noticed they did this in the Dark Souls games, where if you're running around a dungeon you can tell there is an orb of light around the character. We can add this later when building out the level.



# Input System

The game supports Playstation, Xbox, and other generic gamepad controls.

# Version Control




I use Github and git for version in 90% of all cases. However, with this open world game I moved over to Plastic SCM.

I did try LFS with Git, and while the solution can work for large projects, I decided to give Plastic SCM a try, the version control company Unity recently purchased.

I will continue using Github for the majority of projects, but with checking very large files for an open world game Plastic SCM seems like a more dedicated solution. In the future I would also like to try Perforce.

Plastic SCM is built right into Unity, so the workflow is fast. However for smaller projects, I will continue to use git, because git works well for 99% of cases. The only reason I would consider using Plastic SCM is when working with extremely large file sizes, which git was just simply not built for.


![Plastic SCM](https://github.com/pjkw/Portfolio/blob/main/images/Plastic%20SCM.png)


