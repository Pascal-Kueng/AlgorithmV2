# Overview of Changes in the Algorithm

## Introduction of New Features

1. **Ecological Momentary Assessment (EMA)**: A new feature introduced in the updated algorithm. The EMA is a brief survey that is triggered once every day. The timing of the EMA is determined by the response to a question from the previous day's Daily Questionnaire (DQ). If the participants indicate they plan to engage in physical activity, the EMA is triggered before the planned activity slot. If no activity is planned, the EMA is triggered at 12:00. This is done at the individual level, not the couple level.

2. After submitting an EMA, the system should check if it needs to trigger the Second Algorithm to select a situation at this moment or not. 

## Changes to "Determining the Situation"

1. In the pilot study, the appropriate slot (i.e., before planning, before activity, evening) was defined at the intervention level. Now, it is defined at the situation level. We will provide an additional column in the table with all situations, specifying the slot.

2. The selection and triggering process of interventions have been revised.

## Algorithm Adjustments

1. **First Algorithm**:
    - The initial steps remain the same as in the pilot study. However, the formula for the "TotalScore" changes from 2x Severity + Frequency + TimeDelta to 3x Severity + Frequency + TimeDelta.
    - Instead of selecting the situation with the highest score after all TotalScores have been calculated (as was done in the pilot), all TotalScores are logged and the first algorithm ends. The second algorithm is then initiated for the "evening" slot.

2. **Second Algorithm**:
    - Previously, there were two versions: "before planning" and "before activity." Their sole purpose was to check if the situation was present and if a situation was in the slot. Now, they will actually select the situation and the intervention.
    - The second algorithm now aggregates the recent TotalScores from the first algorithm, but only for situations suitable for the current slot. It then uses the "SelectionScore" to calculate the FinalScore and selects the situation with the highest FinalScore.
    - What the SelectionScore is, depends on the slot. The "evening" slot and the "before planning" slot use the last TotalScores as the SelectionScores (today's and the last day's respectively). The "before activity" slot uses the EMA Score. We will provide an additional column to the "Situations" sheet, with a formula based on the EMA questions for each situation. 

## Timing of Slots

1. The "Evening" slot timing remains unchanged, with selection and triggering happening immediately after the first algorithm completes its calculations.

2. The "Before planning" slot is triggered as it was in the pilot study, every Sunday morning.

3. The "Before activity" slot's triggering is now based on a response to a question in the EMA, not on the plans made in the app.

## Intervention Selection

1. Now, this occurs when a version of the second algorithm completes its process, instead of every evening as in the pilot study.

2. The process starts the same as in the pilot until an intervention is selected. Now, there is an additional step where it is randomly determined (with a 50% chance) whether the intervention is triggered. If not triggered, it is logged as selected and scheduled at this time but marked as not triggered.

3. The logic based on the slot of the intervention used in the pilot study is no longer applicable and will be skipped. Instead, the intervention is simply triggered immediately.
