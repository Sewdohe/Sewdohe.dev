---
title: "Create: Blockchain"
description: "A currency-generation solution for the Create: Numismatics mod."
date: 2026-01-11
categories:
  - minecraft
  - java
  - modding
repositoryUrl: https://github.com/Sewdohe/Create-Blockchain
demoURL: ""
status: active
image: "[[currencyminerexample.png]]"
imageAlt: ""
hideCoverImage: false
hideTOC: false
noIndex: false
featured: true
---
Create: Blockchain is a mod about currency generation. It requires both Create: Numismatics and Create: Additions. It's mean to be an economic solution to the Create: Numismatics mod. While Numismatics is a fantastic mod, it provides no way to introduce money into the economy unless you use creative shops. My mod intends to fix that by adding a block that when provided FE (power, not SU from create!) will generate currency at a configurable rate. After X amount of coins are mined, the mining cost will go up by Y (also configurable). Miners will need to be carefully managed by players to get the most of thier generators.

The miner itself is a sided block - takes energy input on either side and outputs coins from the back. Acts as a create vault and items must be pulled out with a funnel or something of the sort. 

## Heat Mechanic
Heat now goes up on the miner block proportional to how much energy the block is fed at any given moment. The block is able to passivley cool itself using just the surrounding air, but to run it at any deccent speeds you'll need to pipe either water or coolant into the top of the miner to keep it at operational temps.

## Custom Coolant
We now have a custom fluid, made by mixing water, ice, and powdered obsidian in a heated basin. This coolant is much more efficent at cooling than water but is much more resource intensive. Can you automate it??

## Mining Cores / Geodes
The currency miner blocks will require a mining core to operate. If no mining core has been inserted, the miner cannot operate. To prevent mass production of miners, I've made the cores have no recipe and spawn inside of custom geodes that can be found rougly evet 30 chunks or so.The mining goes holds durability, and once it reaches 0 the core is no longer any good, and you will have to venture back out to find more cores. Mining cores can also be found in buried treasure chests, and trial chamber vaults.

## Core Finder Tool
You can now craft a special remote that can be charged using the Create: Additions tesla coil, or any other valid FE charger block. Once charged, the remote will send out a search signal and play a tone for the player which lets them know how far/close they are to a mining_core block. This way the cores, while rare, are not impossible to find.

## Piggy Banks
Piggy banks can be found in most standard overworld chests, and drop a random smattering of coins!

## Re-Design
The mining core has been re-designed thanks to EeebsieKats excellent modeling skills, and items have been revampped by the excellent pixel artist DJRaj!!! Big thanks to these two for the assets! Mod wouldn't look nearly as good without them!!!

## Config Values:
- baseEnergyPerCoin - the initial energy amount cost per-coin generated
- difficultyBonus - how much FE is added to the mining cost on each increase 
- difficultyInterval - how many coins must be mined before the difficulty increases
- maxEnergyConsumption - amount of FE miner can consume on each tick 
- energyCapacity - max energy the miner can store
- heatPerFE - heat adder per FE on each tick
- passiveCoolingPercent - amount the miner is air cooled each tick
- waterCoolingPercent - water cooling percent each tick
- cryotheumCoolingPercent - coolant cooling percent each tick


