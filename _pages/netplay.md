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

toc:
  - name: Overview
  - name: What is NetHack?
  - name: NetPlay
  - name: Experiments - Full Runs
  - name: Experiments - Scenarios
  - name: References

_styles: >
  figure {
    margin: 0 0 0 0;
  }
---

<div class="row mt-3">
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

The sheer size of NetHack, paired with the many sub-systems the player must understand, makes it an excellent candidate for evaluating the limitations of LLM agents. We used the NetHack Learning Environment<d-cite key="nle"></d-cite> <b>(NLE)</b> to play NetHack.

## NetPlay
<div class="l-body">
    {% include figure.liquid loading="eager" path="assets/img/nethack_agent.drawio.png" class="z-depth-1" zoomable=true %}
</div>
<div class="caption">
    This is an illustration of how NetPlay plays NetHack. An LLM is prompted to select a skill from a given list using its memory, the current observation, and a task description containing available skills and the desired output format. The chosen skill is executed while a tracker enriches the given observations and detects important events, such as when a new monster appears. The prompting process is restarted when the chosen skill is finished or when an important event occurs to which the agent has to respond.
</div>

Long-term planning in NetHack proves challenging due to its unpredictability, as we cannot know when, where, or what will appear as we explore. Consequently, our agent implements a closed-loop system where the LLM selects skills sequentially while accumulating feedback through game messages, errors, and manually detected events. Although we avoid constructing entire plans, the LLM's thoughts are included for future prompts, allowing for strategic planning if deemed necessary by the LLM.

### Prompting
We prompt the LLM to choose a skill from a predefined list. The prompt comprises three components: <b>(a)</b> the agent's short-term memory, <b>(b)</b> a description of the observation, and <b>(c)</b> a task description alongside the output format.

<b>(a)</b> The observation description primarily focuses on the current level alongside additional data like context, inventory, and the agent’s stats.
We attempt to convey spatial information by dividing the level into structures like rooms and corridors. Monsters are described separately from the structures by categorizing them as close or distant, indicating their potential threat level. The LLM is also informed about which structures can be further explored alongside the positions of boulders and doors that block exploration progress.

<b>(b)</b> The short-term memory is implemented using a list of messages representing the timeline of events. Each message is either categorized as system, AI, or human. System messages convey feedback from the environment, such as game messages or errors. AI messages are used for the LLM's responses. Human messages indicate new tasks. Note that while it is possible for a human to provide continual feedback, we only study the case where the agent is given a task at the start of the game.

<b>(c)</b> The task description includes details about the current task, available skills, and a JSON output format. We employ chain-of-thought prompting<d-cite key="prompting"></d-cite> to guide the LLM to a skill choice.

### Skills

| Name                      | Parameters       | Description                                                      |
| :------------------------ | :--------------| :--------------------------------------------------------------  |
| explore_level             |                  | Explores the level to find new rooms. |
| press_key                 | key: string     | Presses the given letter. |
| pickup                    | x: int, y: int| Pickup things. |
| up                        | x: int, y: int| Go up a staircase. |
| drop                      | item_letter: string| Drop an item.                                                  |
| wield                     | item_letter: string| Wield a weapon.                                                |
| kick                      | x: int, y: int | Kick something.                                                |
| cast                      |                  | Opens your spellbook to cast a spell.                           |
| pay                       |                  | Pay your shopping bill.                                         |

NetPlay uses handcrafted skills that implement different behaviors by returning a sequence of actions. They accept both mandatory and optional parameters as input. Skills can also generate messages as feedback, which are stored in the agent's memory.

Navigation is automated through skills like <b>move_to x y</b> or <b>go_to room_id</b>.
However, exploring levels with only these skills proved challenging for the LLM. To address this, we automated exploration with the <ib>explore_level</b> skill. This skill explores the current level by uncovering tiles, opening doors, and searching for hidden corridors.

To indicate when the agent is done with a given task, it has access to the 
<b>finish_task</b> skill. Additionally, the LLM is equipped with the <b>press_key</b> and <b>type_text</b> skills for navigating NetHack's various game menus.

The remaining skills are thin wrappers around NetHack commands, such as <i>drink</i> or <i>pickup</i>. However, these commands often involve multiple steps, such as confirming which item to drink or positioning the agent correctly to pick up an item. Consequently, the LLM often assumed that the <i>drink</i> command accepts an item parameter or that <i>pickup</i> works seamlessly regardless of the agent's current position. To mitigate these issues, most command-skills automate some aspects to make the skills more intuitive for the LLM.

## Experiments - Full Runs

## Experiments - Scenarios
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

Showcase how NetPlay behaves in different scenarios, what it struggles with, and what we learned in the paper.