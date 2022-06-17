# Slide 1

Hi everyone! Thanks for being here today. I'm really excited to talk about my newest project, unifir: a unifying API for interacting with Unity from R.

# Slide 2

But first, I should introduce myself! I'm Mike Mahoney, and I'm a PhD student at the State University of New York College of Environmental Science and Forestry. 

My work really focuses on how we can help people understand large-scale systems, especially natural systems like ecosystems and forests. You see, when you talk about a forest, you're talking about something that develops over an entire landscape over the course of hundreds of years, which means that you can't really observe the entire system in the span of a single human life. That makes it really hard to develop an intutive understanding of these systems! But we need to make decisions about what we do with our forests -- what trees we plant, what harvesting we can do without destroying the system, what areas to prioritize conserving -- even without being able to watch the outcomes of any of those decisions from beginning to end. 

And so my group has been looking at ways that we can use visualization to try and help build those intuitive understandings so that it's easier to reason about and manage these large-scale systems. And in particular, we've been looking at using video game engines as a way to make those visualizations.

# Slide 3

So for instance, last year some people in my lab published this paper, "Using tree modeling applications and game design software to simulate tree growth, mortality, and community interaction". And what they did was use this program, the Unity video game engine, to render this virtual forest that's on screen. And in the game engine, we can fast forward and move through time to show how trees compete for resources, how less competitive trees slowly die off, and really show how competition impacts the development of a forest over time. We can collapse this system into a form that humans can understand.

# Slide 4

And similarly, last year I had this paper out on "Interactive Landscape Simulations for Visual Resource Assessment". And the idea here is that in the middle of this image is a red dot, what the arrow is pointing at, and all the light up areas around it are in its "viewshed" meaning they can see and be seen by that dot. And viewsheds are nothing new, it's an algorithm that ships in most GIS systems, but by bringing it into the game engine we can let users actually walk across the landscape and start interacting with the outputs from the algorithm. By bringing this into the game engine, we can start helping people build a more intuitive understanding of how visibility interacts with topography, and also validate our algorithm by going and standing at the locations we think it might have got wrong.

# Slide 5

Both of these examples use the Unity 3D video game engine to build their environments. And Unity is a fantastic tool -- it can render huge landscapes with tons of 3D models, has interactivity built in, and does everything a game developer could want.

But we're not game developers in my lab, and so we've run into some challenges. Unity makes it really hard to make your environments in an open and reproducible manner. Usually, game environments are being created by hand by a designer, and they don't need to be easily re-created from scratch. And because the environment was built by hand, by a designer, there's no record of the active decisions that were made while building the environment, making it even harder to independently reproduce.

Plus, Unity doesn't really "speak" scientific data. There's no real support for spatial data formats, and no easy way to control the size and shape of objects in the environment based on data inputs. That can make it really hard to visualize the outputs from scientific tools, or just to visualize real-world spaces.

And last but not least, Unity is complicated. A lot of interactions with Unity can be done through the user interface, which is decently complex and doesn't scale to working with a large number of objects. If you want to be more efficient, you can write scripts to work with Unity directly, but Unity only speaks C sharp on the backend. And most researchers don't know C sharp, so that's a real limitation to using Unity for a lot of visualization projects!

# Slide 6

So last year at useR I spoke about our first stab at dealing with some of these problem, with the terrainr package. And terrainr does a few things -- it's an API client for the United States Geological Survey's National Map, it's got some 2D visualization stuff in it -- but the core feature, in my mind, is that it handles turning terrain data into something Unity can deal with.

# Slide 7

For instance, I won't get too deep into the code here, but this is 12 lines of code all told. And this 12 lines gets a centroid for Zion National Park, in the southwestern USA, downloads elevation data and orthoimagery for an eight kilometer square around that point, and then turns the data into a format that can be brought into Unity. 

# Slide 8

All the user needs to do is drag the files into a Unity project, click a single button, and presto, they've got a terrain with imagery in their game engine.

And this was big for us! This enabled a lot of work we hadn't been doing before, because we could now bring in all sorts of spatial data into Unity. Before terrainr, there wasn't a good way to add imagery to these surfaces, or to represent vector data like points or polygons on your terrain.

# Slide 9

But to go further than that -- to, for instance, actually add 3D models, or add a player character and walk across the terrain -- you still needed to know Unity, and you still needed to know C#. terrainr could handle all the data processing for you, and could give you exactly what Unity expected, but you still needed to get over that final hump.

# Slide 10

And so, since that last conference I've been working on a new package, called unifir.

unifir is meant to be a unifying API for interacting with Unity: it's supposed to be the low-level plumbing that packages can rely on to create and manipulate environments in Unity. 

# Slide 11

The way it does that is through objects which I call "props". Users create "props" through a handful of normal, idiomatic R functions -- provided by either unifir or an extension package. And those functions produce an R6 object that captures the input that a user gave, as well as instructions for how to translate that R code into C sharp. 

So here, for instance, I called the R function that you'd use if you want to add a light to your Unity environment. And all the arguments we pass through R get stored in the "parameters" slot of this object, and will be translated into C sharp by the "build" function that the prop creates. But as a user, you don't need to know or think about any of that; you can write relatively normal R code and expecting things to just work under the hood.

# Slide 12

So then those props are then in turn stored in a "script" object. And a script is another R6 object, which is basically responsible for tracking what props a user has created and in what order. So there's a lot of objects contained in each script, including each prop and the instructions for how to translate it into C sharp, a list of what order the props should run in, and some other configuration options. And the nice thing here is that a user doesn't need to keep track of their prop objects, they're always working on a single script, the same way you'd expect to work with a data.frame or other object in R.

But also, because a script contains a record of all the other props a user has already added, props are able to actually edit themselves and each other based on what's already in the script. And that lets us write really efficient props, by changing prop behavior based on what else the script is supposed to be doing. For instance, to create a 3D model, you need C sharp code that will load the 3D model, save it to the right place in your Unity project, create it as an object in Unity, move it to the right location, resize it, and so on. And if you went through that entire process for each model you want to add, creating things like forests would take a really long time. This ability for props to inspect the rest of the script means we can reuse C sharp code between props, which makes the actual scene creation a lot faster.

# Slide 14 

And so once a user has created all of their props and is ready to create their scene, they can use the "action" function to actually execute their script. And when you do that, unifir will step through each of the props you've added in order, follow the instructions in them to write C sharp code, and then execute that resulting C sharp inside of Unity itself. And so entirely from R we're able to write these complicated programs that create 3D environments, in Unity, without needing to use the graphical interface at all.

# Slide 15

And so just for example's sake, this is what one of these unifir programs can look like. This is a really simple example, which is generating a bumpy terrain surface and placing a bunch of trees at random positions on the surface.

# Slide 16

And running that code in R produces a Unity environment automatically, no input from us required.

# Slide 17

So that's what unifir provides itself, directly. But it also provides the ability for packages to write their own props, so that they can rely on unifir's approach for interacting with Unity and not need to worry about dealing with the API themselves.

So for instance, here's the terrainr code from the start of this talk. And remember, this code downloads a bunch of spatial data and turns it into something you can bring into Unity yourself, but doesn't handle the interaction with Unity at all.

# Slide 18

Well, the new version of terrainr on CRAN now wraps unifir. And so we can replace that make manifest call with make unity,

# Slide 19

and that will go ahead and create our terrain surface inside a new Unity project for us. No human interaction required.

# Slide 20

And, if we write a little bit of unifir code ourselves, we can actually add a player on top of this terrain.

# Slide 21

And so we now have fully interactive 3D landscape visualizations, in Unity, created entirely in R. 

And this is a huge step forward for reproducibility and ease-of-use in our work. Especially with the push towards more and more open science practices, being able to show people a clear expression of the decisions we made when making an environment is huge. 

# Slide 22

And so that's unifir! Over the next few months we've got a few more features we'd like to add -- we want to wrap more of the Unity API, and add some features so that it's easier to procedurally generate landscapes directly from R and port those into Unity. But mostly, we're excited to finally have this plumbing for our own work, so we can start making these environments faster and more openly for our future studies!

# Slide 23

And that's just about my time! I want to thank the ESF Pathways to Net Zero Carbon initiative for funding this work. You can find me on everything at mikemahoney218, and these slides are on GitHub at mm218.dev/user2022. Thanks again!
