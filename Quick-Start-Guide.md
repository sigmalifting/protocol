<!--
SigmaLifting Protocol - Quick Start Guide
Copyright (c) 2025 SigmaLifting LLC
Licensed under CC BY 4.0: https://creativecommons.org/licenses/by/4.0/
-->

# Quick Start Guide

## Core Concepts

### Programs vs Processes

**Programs** are training templates you design once and reuse. They contain the structure and logic of your training plan.

**Processes** are running instances of programs where you log your actual workouts. You can run the same program multiple times, each with its own logged data.

### Blocks

A **block** is consecutive weeks with an identical day-by-day layout—same training days, same exercises for each day.

This constraint is fundamental to the app's design. By fixing weekly structure within a block, the app captures your training cleanly and straightforwardly—you focus purely on how values progress week-to-week while all underlying calculations take care of themselves. When your training requires a different layout, simply start a new block.

You arrange multiple blocks sequentially to create a complete program.

### Set Groups

A **set group** is a collection of sets that are identical from a programming perspective. Instead of programming 3 sets of 5 reps @ rpe 6 as three separate sets, you create one set group with 3 sets. All sets in a group share the same structure and prescribed values.

### Variable Parameter

Each set group has one **variable parameter** - the value you record and can only be determined during training. The other two parameters are pre-programmed (prescribed).

- If **weight** is variable: You record the weight used. Reps and RPE are pre-programmed.
- If **reps** is variable: You record the reps completed. Weight and RPE are pre-programmed.
- If **RPE** is variable: You record the RPE felt. Weight and reps are pre-programmed.

### Backoff Sets

**Backoff sets** calculate their weight based on a previous set group's performance. When configuring a backoff set, you specify which earlier set group it references. Since a set group can contain multiple sets, the calculation uses the last set of the source set group.

Example: If Set Group 2 backs off from Set Group 1, it uses the last set of Set Group 1's recorded weight for its calculation.

### Fatigue Drop

**Fatigue drop** applies only to set groups where RPE is the variable parameter. When you record an RPE that reaches or exceeds the RPE cap, the system automatically reduces the prescribed weight.

The **RPE cap** is an upper limit that triggers the fatigue adjustment. When your RPE equals or exceeds this value (≥), the fatigue drop activates.

### 1RM Anchor

The app has no exercise library—exercise names are arbitrary strings you define. **1RM anchoring** creates the mathematical relationships needed for percentage-based weight calculations.

In a process, you enter 1RM values for squat, bench, deadlift, and any custom lifts defined in the program. Any exercise can be anchored to these lifts with a percentage ratio.

Example: A front squat anchored to squat at 85% means when your squat 1RM is 200kg, the front squat calculations use 170kg as its reference. You can also define custom lifts (like "OHP" or "Front Squat") and anchor other exercises to them.

### Simple Deload

**Simple deload** is a special case for deload weeks. When enabled, the final week of a block calculates its weight as a percentage of the recorded weight from the week before. This automates the common pattern of reducing intensity in the last week while basing it on recent performance.

### Weight Calculations

Prescribed weights can be calculated as:

- Percentage: Based on the exercise's 1RM
- Absolute: Fixed weight values
- Mixed: Combination of Percentage and Absolute as offsets
- Backoff: Percentage of another set group's recorded weight
- Fatigue Drop: Adjusted based on RPE thresholds
- Simple Deload: Percentage of the recorded weight from the week before

When weight is the variable parameter, the app suggests weight range based on 1RM, reps and RPE.
