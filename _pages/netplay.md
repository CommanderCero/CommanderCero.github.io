---
layout: distill
title: NetPlay
tags: nethack llm zero-shot game-playing
date: 2024-04-04
permalink: /netplay/
classes: wide

bibliography: netplay.bib

authors:
  - name: Dominik Jeurissen
    affiliations:
      name: Queen Mary University of London
  - name: Diego Perez Liebana
    affiliations:
      name: Queen Mary University of London
  - name: Jeremy Gow
    affiliations:
      name: Queen Mary University of London

_styles: >
  figure {
    margin: 0 0 0 0;
  }
---

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dummy_600_400.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Overview
Recently, Large Language Models <b>(LLMs)</b> have shown great success as high-level planners for game-playing agents. However, many of these agents are primarily evaluated in simple environments with few objects or interactions<d-cite key="innermonologue"></d-cite><d-cite key="deps"></d-cite>, or in games such as Minecraft<d-cite key="gitm"></d-cite><d-cite key="voyager"></d-cite>, where long-term planning is relatively straightforward.

We introduce NetPlay, the first LLM-powered zero-shot agent for the challenging roguelike NetHack<d-cite key="nethack"></d-cite>. Building upon existing approaches tailored for simpler dynamic environments such as Inner Monologue<d-cite key="innermonologue"></d-cite>, we extended its capabilities to address NetHack's complexities. We evaluated the agent’s ability to play the full game and analyzed its behavior using various isolated scenarios.

NetPlay excels in executing detailed instructions but struggles with more ambiguous tasks, such as winning the game. We discovered that NetPlay’s strength lies in its flexibility and creativity. Our experiments show that given enough context information, NetPlay can perform a wide range of tasks. Moreover, by focusing its attention on a particular problem, NetPlay is adept at exploring a wide range of potential solutions, but often with limited success due to a lack of explicit feedback guiding it.

## What is NetHack?
<div class="l-body">
    {% include figure.liquid loading="eager" path="assets/img/nethack_example.png" class="z-depth-1" zoomable=true %}
</div>
<div class="caption">
    A screenshot of NetHack's terminal view. Source <a href="https://alt.org/nethack">alt.org/nethack</a>.
</div>

NetHack<d-cite key="nethack"></d-cite>, released in 1987, is an extremely challenging turn-based roguelike that continues to receive updates to this date. The objective is to traverse 50 procedurally generated levels, retrieving the Amulet of Yendor and successfully returning to the surface. Doing so unlocks the final challenge of the game: the four elemental planes, followed by the astral plane, where players must present the Amulet to their deity.

NetHack encompasses a diverse array of monsters, items, and interactions. Players must skillfully utilize their resources while avoiding many of the game's lethal threats. Even for seasoned players possessing extensive knowledge of the game, victory is far from guaranteed. The game's inherent complexity requires players to continuously re-assess their situation to adapt to the unpredictability of the elements at play.

The sheer size of NetHack, paired with the many sub-systems the player must understand, makes it an excellent candidate for evaluating the limitations of LLM agents. Note that we used the NetHack Learning Environment<d-cite key="nle"></d-cite> <b>(NLE)</b> to play NetHack.

## NetPlay
A showcase of prompting NetPlay and how it works

## Scenarios
Showcase how NetPlay behaves in different scenarios, what it struggles with, and what we learned in the paper.