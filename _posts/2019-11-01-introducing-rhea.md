---
layout: post
title: Introducing Rhea
description: A tick-based 4x space conquest strategy MMO
---

![](https://s3.amazonaws.com/images.willricketts.com/rhea/scifi.jpg)

Over the last several months, I've been working on the backend for a lofty project. While not _that much_ of a PC gamer, I generally focus on two types of games:

### Grand Strategy / 4x (explore, expand, exploit, exterminate)
  - [Europa Universalis 4](https://www.paradoxplaza.com/europa-universalis-all/)
  - [Hearts of Iron](https://www.paradoxplaza.com/hearts-of-iron-iv/HIHI04GSK-MASTER.html)
  - [Stellaris](https://www.paradoxplaza.com/stellaris/STST01G-MASTER.html)
  - [Crusader Kings 2](https://www.paradoxplaza.com/crusader-kings-ii/CKCK02GSK-MASTER.html)

  <br/>

### Sci-fi Sandboxes
  - [Eve Online](https://www.eveonline.com/)
  - [the idea of] [Star Citizen](https://robertsspaceindustries.com/) (lol if it ever comes out)
  - [Planetarion](http://planetarion.com/)

  <br/>

Aside from Star Citizen, due to the possibility of it being vaporware, all of these titles have been staples of the last several years of PC gaming for me-- especially Eve Online, Planetarion, and Stellaris.

Each of these games have unique characteristics that I really enjoy, but I've always thought about what it would be like for a game to incorporate all of them in harmonious union. Eve Online's metagame, political intrigue, complex player-driven economy, and expansive universe create an immersive third person experience with an extremely high skill cap. Stellaris focuses more on developing and scaling an empire from the perspective of its leader. Planetarion shares with Stellaris its perspective, but is a tick-based game played largely in text within a web browser, making it extremely portable and not totally married to a client with its own specific system or platform needs. To distill these properties into those that inspire the dynamics of Rhea:

### Eve Online
  - Persistent universe
  - Player-to-player political intrigue
  - Player-driven economy
  - Robust industrial system

  <br/>

### Stellaris
  - Played from the context of the leader of an empire as opposed to Eve's perspective of a single pilot in a large universe
  - Player-driven relationships with NPC factions

  <br />

### Planetarion
  - Can be played in the browser
  - Tick-based

  <br/>

Whew. There's a lot to unpack there. Suffice to say, I've had a bit of difficulty describing the project in a brief way that's easy to understand for someone who's unfamiliar with these three titles, so I've taken to describing Rhea as "a tick-based 4x space conquest strategy MMO." Most people are familiar with terms like "MMO" and "space conquest," though a bit of this might take some explanation.

The tick system of Planetarion is very different than the temporal systems of most strategy games, in that a "tick" takes place every hour. A tick consists of all of the calculations required to update the game's state for the next hour. Player resource balances are updated, fleets are moved based on their course, and combat outcomes are calculated. This allows the game to be played over a longer period of time in which the players are able to set up actions to be performed, and the system will execute them over time. The player is able to note when their attention will be needed next, and can return to the game upon a tick in which they want to perform their next action.

## The Game

#### Map

Rhea's map is in essence, a giant DAG (directed acyclic graph) with solar systems represented as nodes and the stargates that connect them represented as the edges between those nodes.

![](https://s3.amazonaws.com/images.willricketts.com/rhea/dag.png)

This is not unlike the maps of all three games I've used as examples and inspiration for Rhea. In my view, it's a great system, so I'll borrow from their developers' wisdom. The interesting difference in Rhea is that the Galaxy, in its current working model, contains 2000 distinct solar systems.

Check out an _extremely_ early-stage rendering of Rhea's game map [here](https://master.d1nj6eczclst4t.amplifyapp.com/map). Also to note is that this is not the final structure of the galaxy, as it's simply the initial result from being fractally generated. There's still plenty of work to be done in transforming the structure to have more interconnectedness and to be less of a corridor and chokepoint heavy layout.

#### The Player's Position in the Galaxy

When the player creates a character and enters the game, as of the time of this writing, they are provided a basic ship, are given some basic resources for expanding their wealth and influence, and are dropped into the game in a central area of the map in which they cannot be attacked by other players. This is a concept that was inspired by Eve Online's idea of "high security space" or "hisec." This is essentially an area of the game that's PVE (player versus environment) only in which attacks on other players are disabled. The player has the freedom to exploit the very basic resources in this area and eventually equip themselves to venture further away from the galactic center in pursuit of greater wealth or conquest.

Players are able to claim space as their own, colonize planets, build facilities on orbital bodies that exploit their resources, and build defensive structures to stave off attacks and raids from other players or groups of players.

Players can form organizations with other players, are able to claim solar systems on their behalf, and can go to war with other player organizations for any number of reasons-- from conquest and annexation of their enemies' sovereign space to a simple raid over an enemy's borders to pillage resources from an orbital facility.


## The Codebase

I've chosen Elixir for the backend implementation of the various game services due to its wonderful ability to handle i/o bound operations with great efficiency. I also believe using a functional language is wise for building a "tick" system as a tick itself is basically a process of performing a large mutation of externally held state (PostgreSQL) in a predictable and repeated way.

#### Structure

Rhea's backend, in its current state, is built as an Elixir umbrella with a few sub-apps handling various aspects of the game, such as its public api, persistence layer, galactic map (more on this part of the project in later posts), and a general orchestration layer, `main`.

Aside from `main`, each of these sub-apps exposes an interface to its various public functions as to prevent cross-application coupling:

```elixir
defmodule GalaxyMap.Interfaces.Importers.MapGraph do
  @moduledoc """
  Interface for Map Graph importer
  """
  alias GalaxyMap.Importers.MapGraph

  defdelegate run(solar_systems), to: MapGraph
  defdelegate clear_graph_data(), to: MapGraph
end
```

#### Temporal Features

Rhea leverages the [Quantum](https://github.com/quantum-elixir/quantum-core) library to handle its various timed jobs, the majority of which take place when a tick is executed (every 20 minutes). Other various temporal features include scheduled statistics gathering, fleet combat precalculation, etc.


## Wrapping Up

Rhea is a large project. For a single developer working on it in their spare time, it's a _really_ large project, but it's a game that combines my favorite mechanics of my favorite titles as well as adding mechanics and systems I've always wanted to see in the 4x strategy genre. In essenece, I feel like I'm building the game I've always wanted.

I plan on releasing more comprehensive posts regarding game mechanics, technical implementation, and game lore in the future on this blog. Thanks for taking a look :)