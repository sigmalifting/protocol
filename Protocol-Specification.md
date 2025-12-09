<!--
SigmaLifting Protocol Specification
Version: 1.0.0
Copyright (c) 2025 SigmaLifting LLC
Licensed under CC BY 4.0: https://creativecommons.org/licenses/by/4.0/
-->

# SigmaLifting Protocol Specification

## Overview

SigmaLifting is a data protocol for representing powerlifting training data. It defines a structured format for program templates, running instances, and exercise prescriptions that can be implemented across different platforms while maintaining data integrity and portability.

## Data Model Architecture

The protocol consists of three main components:

1. **Programs**: Reusable training templates
2. **Processes**: Running instances with workout logs
3. **Exercises**: Exercise definitions with progression logic

### Entity Relationships

```
Program (1) ──── has many ────> (N) Blocks
Block (1) ──── has many ────> (N) Training Days
Training Day (1) ──── references ────> (N) Exercises
Exercise (1) ──── has many ────> (N) Set Groups
Process (1) ──── instantiates ────> (1) Program
Process (1) ──── has many ────> (N) Exercise Recordings
```

## Core Schemas

### Program

The root entity representing a training template.

```typescript
interface Program {
  _id: string; // Unique identifier
  name: string; // Display name
  description: string; // Program description
  created_at: string; // ISO 8601 datetime
  updated_at: string; // ISO 8601 datetime
  blocks: Block[]; // Sequential training blocks
  custom_anchored_lifts: string[];
}
```

### Block

A training phase with a fixed weekly structure. Duration is adjustable.

```typescript
interface Block {
  _id: string;
  name: string;
  program_id: string; // Parent program reference
  duration: number; // Weeks
  weekly_schedule: string[]; // Array of 7 training day IDs or empty strings
  training_days: TrainingDay[]; // Training day definitions
}
```

### Training Day

A workout session template within a block.

```typescript
interface TrainingDay {
  _id: string;
  name: string;
  exercises: string[]; // Ordered array of exercise IDs
}
```

### Exercise

Complete exercise definition with progression parameters.

```typescript
interface Exercise {
  _id: string;
  program_id: string;
  block_id: string;
  exercise_name: string;
  one_rm_anchor: OneRmAnchor;
  deload_config?: DeloadConfig;
  set_groups: SetGroup[]; // Max 15 groups
  created_at?: string;
  updated_at?: string;
}
```

### Set Group

A collection of identical sets with shared parameters and progression logic.

```typescript
interface SetGroup {
  group_id: string;
  variable_parameter: "weight" | "reps" | "rpe";

  // Weekly arrays - length must match block duration
  weekly_notes: string[];
  weekly_num_sets: number[]; // Sets per week
  weekly_reps: number[];
  weekly_rpe: number[];
  weekly_weight_percentage: number[];

  // Optional configurations
  mix_weight_config: MixWeightConfig;
  backoff_config: BackoffConfig;
  fatigue_drop_config: FatigueDropConfig;
}
```

### Process

A running instance of a program with actual workout data.

```typescript
interface Process {
  _id: string;
  name: string;
  program_id: string;
  program_name: string;
  user_id?: string;
  created_at: string;
  updated_at: string;
  start_date: string; // ISO 8601 date
  config: ProcessConfig;
  one_rm_profile: OneRmProfile;
  exercise_recordings: ExerciseRecording[];
}
```

### Exercise Recording

Logged workout data for a specific exercise within a process.

```typescript
interface ExerciseRecording {
  exercise_id: string;
  weekly: WeeklyRecord[]; // Length matches exercise's block duration
}

interface WeeklyRecord {
  sets: SetRecord[];
  note: string;
}

interface SetRecord {
  weight?: number;
  reps?: number;
  rpe?: number;
  completed: boolean;
}
```

## Configuration Objects

### One RM Anchor

Links exercise weights to powerlifting competition lifts.

```typescript
interface OneRmAnchor {
  enabled: boolean;
  lift_type?: string;
  ratio?: number;
}
```

### Mix Weight Configuration

Allows combining percentage-based and absolute weight adjustments.

```typescript
interface MixWeightConfig {
  enabled: boolean;
  weight_unit?: "kg" | "lbs";
  weekly_weight_percentage?: number[];
  weekly_weight_absolute?: number[];
}
```

### Backoff Configuration

Defines weight calculation based on previous sets.

```typescript
interface BackoffConfig {
  enabled: boolean;
  depends_on_set_group_id?: string; // Must reference earlier set group
  type?: "%_of_weight" | "target_rpe" | "";
  weekly_rpe?: number[]; // For target_rpe type
  weekly_percentage?: number[]; // For %_of_weight type
}
```

### Fatigue Drop Configuration

Applies RPE-based adjustments when RPE is the variable parameter.

```typescript
interface FatigueDropConfig {
  enabled: boolean;
  rpe_cap?: number[]; // RPE threshold (≥ triggers drop)
  type?: "%_of_weight" | "target_rpe" | "";
  weekly_rpe?: number[];
  weekly_percentage?: number[];
}
```

### Deload Configuration

Enables automatic deload calculation for the final week of a block.

```typescript
interface DeloadConfig {
  enabled: boolean;
  percentage?: number;
}
```

### Process Configuration

User preferences for a specific process instance.

```typescript
interface ProcessConfig {
  weight_unit?: "kg" | "lbs";
  weight_rounding?: number;
  week_start_day?: number;
}
```

### One RM Profile

Tracks 1RM values for the main lifts across the process.

```typescript
interface OneRmProfile {
  enable_blockly_one_rm: boolean;
  squat?: number;
  bench?: number;
  deadlift?: number;
  blockly_one_rm?: BlocklyOneRm[];
  [custom_lift: string]: number | undefined;
}

interface BlocklyOneRm {
  squat?: number;
  bench?: number;
  deadlift?: number;
  [custom_lift: string]: number | undefined;
}
```

## Field Constraints

### Special Values

- **-1**: Used for all unset/null numeric fields
- **Empty string**: Used for unset string references

### String Lengths

- Program/Block/Exercise names: max 80 characters
- Description: max 1000 characters
- Notes: max 500 characters
- IDs: max 200 characters

### Array Size Limits

- Blocks per program: max 15
- Training days per block: max 100
- Set groups per exercise: max 15
- Custom anchored lifts per program: max 10 (including squat/bench/deadlift)
- Sets per group: 0-15
- Weekly schedule: always 7 days (fixed)

### Numeric Ranges

- Weight: 0-2000
- Weight delta: -2000 to 2000
- Reps: 0-50
- RPE: 1-10
- Percentage: 0-200%
- Anchor ratio: 0-150%
- Block duration: 1-15 weeks
- Weight rounding: 0.1-50
- Week start day: 0-6 (Sunday-Saturday)

### File Size Limits

- Maximum JSON import size: 5MB
- Maximum file size for export: No explicit limit (but constrained by array limits)

### Array Length Rules

All weekly arrays within a set group must have length equal to the parent block's duration:

- `weekly_notes`
- `weekly_num_sets`
- `weekly_reps`
- `weekly_rpe`
- `weekly_weight_percentage`

### Reference Integrity Rules

1. Exercise `block_id` must match the block containing its training day reference
2. Backoff dependencies must reference set groups that appear earlier in the same exercise
3. All exercise IDs in training days must have corresponding exercise definitions
4. Process recordings must exist for all exercises referenced in the program

## Import/Export Formats

### JSON Format

The protocol defines a standard JSON structure for data portability:

```typescript
interface ProgramExport {
  program: Program;
  exercises: Exercise[];
  metadata?: {
    exportedAt: string; // ISO 8601 datetime
    version: string;
    appName: string;
  };
}

interface ProcessExport {
  process: Process;
  program: Program;
  exercises: Exercise[];
  metadata?: {
    exportedAt: string; // ISO 8601 datetime
    version: string;
    appName: string;
  };
}
```

### Excel Format

The Excel format creates a workbook with the following worksheets:

#### Block Worksheets (One per block)

Each block gets its own worksheet named "Block N - [Block Name]". The layout arranges weeks horizontally and days vertically:

```
+------------------+------------------+------------------+
|     Week 1       |     Week 2       |     Week 3...    |
+------------------+------------------+------------------+
| Day 1            | Day 1            | Day 1            |
|   Exercise 1     |   Exercise 1     |   Exercise 1     |
|     Set 1        |     Set 1        |     Set 1        |
|     Set 2        |     Set 2        |     Set 2        |
|   Exercise 2     |   Exercise 2     |   Exercise 2     |
|     ...          |     ...          |     ...          |
+------------------+------------------+------------------+
| Day 2            | Day 2            | Day 2            |
|   Exercises...   |   Exercises...   |   Exercises...   |
+------------------+------------------+------------------+
| Day 3            | Day 3            | Day 3            |
|   Exercises...   |   Exercises...   |   Exercises...   |
+------------------+------------------+------------------+

                    Weekly Schedule →
```

##### Exercise Grid Layout

Each exercise displays sets in rows with these columns:

```
set    |---------- Prescription ----------|      |---- Execution ----|
no.    weight  reps  rpe  attributes  notes    weight  reps  rpe  notes
--------------------------------------------------------------------
1              5     8                         102.5   5     7
2              5     8                         102.5   5     7.5
```

Special markers:

- Week/Day headers: `$ Week 1 - Day 1 - Upper Body`
- Exercise headers: `# Squat (max: 85% squat, set groups: 3+2)`
- Weekly schedule: `-> Weekly Schedule`

#### 1RM Profile Worksheet

Contains:

- Configuration (weight unit, rounding, week start day)
- Global 1RM values (Squat, Bench, Deadlift)
- Block-specific 1RM values (if enabled)

#### RPE Chart Worksheet

Standard RPE percentage chart (RPE 6-10 × Reps 1-9)

#### Metadata Worksheet

Export information including program name, process name, export date, and version
