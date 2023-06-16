# AlgorithmV2: Time and Ties Project Update  
Updated Algorithm for Time and Ties Project  
We use this algorithm to select the best intervention based on i) long term responses to the daily questionnaire to detect tendencies and patterns in the couples, and ii) shortly before sending an intervention to recognize a situation adequate for intervening.  

Broad Overview:  
<img src="https://github.com/Pascal-Kueng/AlgorithmV2/assets/108430531/077bc429-9f06-4cc6-b1ad-69205e6064e6" width=750>


Specific changes from the <a href="https://github.com/Pascal-Kueng/AlgorithmV2/blob/main/Algorithm%20Pilot.png">old algorithm</a> are summarized <a href="https://github.com/Pascal-Kueng/AlgorithmV2/blob/main/changes_algorithm.md">here</a>.  
Following in this document is a complete summary of the new algorithm.  

## Complete System Summary (new algorithm)

### Ecological Momentary Assessment (EMA) Triggering and Scoring

The system should include a mechanism for triggering an Ecological Momentary Assessment (EMA) close to when we expect a physical activity to be performed. EMAs are brief surveys that ask individuals about their experiences, behaviors, and moods at the moment.  

The triggering of an EMA should occur exactly once every day and can happen at different specific times throughout the day (5am, 11am, 12pm, 1pm, and 5pm).  

The decision to trigger an EMA should be based on the individual's expected activity for the day, which can be retrieved from their last daily questionnaire. If no activity is planned, the EMA should be triggered during the lunch time slot (12pm). If an activity is planned for a specific time slot, the EMA should be triggered during that time slot.  

In the EMA, the person will again specify whether and in which slot they expect to be active. When the EMA is submitted, the system should check if an expected activity is still upcoming for the day. If it is, the system should start the selection of a situation for 'before_activity' right away (or even better, as soon as the expected time of activity has arrived) (see Algorithm 2).  


Flowchart EMA:  
<img src = "https://github.com/Pascal-Kueng/AlgorithmV2/assets/108430531/a110a257-84af-4241-8f05-e94995b0784a" width=750>


Pseudo code example implementation for EMA Triggering and Scoring:

```javascript
let was_already_triggered = false 

// Cronjob:
function decide_if_trigger_EMA(person, upcoming_time_slot, was_already_triggered) {
    if (was_already_triggered) {
        if (upcoming_time_slot === 4) { // last slot of the day, reset this variable
            was_already_triggered = false
        }
        return null
    }

    let expected_activity = get_answer_from_last_daily_questionnaire(person)['ss_exp_active'] // retrieve this persons last daily questionnaire answers. 
    
    if (expected_activity === 0) { // 0 === no activity planned
        if (upcoming_time_slot === 88) { // lunch-EMA time slot
            trigger_EMA() // send EMA to person. See logic in EMA_submitted() below as well. 
            was_already_triggered = true
            return null
        } else {
            return null
        }
    }

    if (expected_activity === upcoming_time_slot) { 
        trigger_EMA()
        was_already_triggered = true
    }

    return null
}


// The actual EMA should also include some logic:
// when EMA was submitted by "person":
// alternatively this could be handled with a separate cronjob that checks in time-intervals, if the EMA was submitted (like Algorithm 1 below).
function EMA_submitted(person, EMA_answers, upcoming_time_slot) {
    let expected_activity = EMA_answers['ss_exp_active']

    if (expected_activity >= upcoming_time_slot) { // greater or equal, so if the activity is upcoming today still. 
        select_situation('before_activity') // ideally, trigger at the expected_activity time!
    }
    return null
}
```

### Algorithm 1: Calculate TotalScores

The system should include a mechanism for calculating TotalScores based on the responses to the daily diaries. This is the score defining which of our pre-defined situations matches the couple's existing condition best, while taking frequency and time delta into account (avoiding overselecting the same situation). Like in the pilot study, the system can execute this algorithm every 5 minutes between 6pm and 5am for each couple. The calculation should only proceed if both partners have completed their diaries. If not, and it's not yet past 4am, the system should wait until the next check. Otherwise, it should log all total scores as zero and trigger situations 0A, 0B, and 0C.

The total score for each situation should be calculated as a combination of (3x) severity score, frequency score, and time delta. These scores represent the severity of the situation, how often it occurs, and the time since it last occurred, respectively.

Flochart Algorithm 1:  
<img src="https://github.com/Pascal-Kueng/AlgorithmV2/assets/108430531/f0b897cc-46bc-43fb-922c-48be3d1eb88b" width=500>

Pseudo code for Algorithm 1: Calculate Situation Scores:

```javascript
// Selection Cronjob: 
// every 5 minutes between 6pm and 5am (foreach couple):
function daily_situation_selection(couple) {
    if (!is_partner_A_diary_complete or !is_partner_B_diary_complete) {
        
        if (!is_past_4am) {
            return null // check again at next cronjob
        } else {
            trigger_situations_0A_0B_0C() // same as pilot
            log_all_total_scores_as_zero() // in same files as in pilot (selection chains etc.)
            return null
        }
    }

    calculate_total_scores()
    select_situation('evening')
}


function calculate_total_scores(all_situations) {
    let total_scores = {} // key: situation, value: score
    foreach situation in all_situations {
        let SeverityScore = calculate_severity_score(situation) // same as pilot. IMPORTANT: If variables are missing, use NA, not 0. 
        let FrequencyScore = calculate_frequency_score(situation) // same as pilot.
        let TimeDelta = calculate_time_delta(situation) // same as pilot

        TotalScore = 3 * SeverityScore + FrequencyScore + TimeDelta
        total_scores[situation] = TotalScore
    }

    log_total_scores(total_scores)
    return null
}
```

### Algorithm 2: Select Situations

The system should include a mechanism for selecting situations. This should be done every Sunday at 4am for each couple (slot “before planning”), but also when triggered by an EMA (slot “before activity”) or by Algorithm 1 (slot “evening”).

The system should calculate the AggregatedScore, which is the mean of the last 14 TotalScores. Then, a SelectionScores is retrieved, which is different depnding on the slot. For the slots "evening" and "before planning" the SelectionScores are the TotalScores form the last daily questionnaires. For the slot "before activity", the SelectionScores are the EMAScores. 

Then, the FinalScores for each situation should be calculated as the weighted geometric mean between the AggregatedScores and the SelectionScores. The situation with the highest FinalScores is selected.

Flowchart Algorithm 2:  
<img src="https://github.com/Pascal-Kueng/AlgorithmV2/assets/108430531/1566da2d-4118-482c-ae66-29dde8eda423" width=500 >

Pseudo code for Algorithm 2: Select Situations:

```javascript
// Cronjob:
// sundays at 4am (foreach couple):
// can also be triggered by EMA (see above) and other Algorithm 1. 
function select_situation(slot) {
    let situations_for_current_slot = get_situations_for_current_slot(slot) // simple subsetting

    final_scores = {}
    foreach (situation in situations_for_current_slot) {
        let last_14_total_scores = get_last_14_total_scores(situation) // from log files somehow. 

        let AggregatedScore = mean(last_14_total_scores)
        
        let SelectionScore
        if (slot === 'evening' or slot === 'before_planning') {
            SelectionScore =last_14_total_scores[-1] // last Total Score from DQ
        } else if (slot === 'before_activity') {
            SelectionScore = get_todays_EMA_score(situation) 
        }

        FinalScore = (AggregatedScore^0.6) * (SelectionScore^0.4)
        final_scores[situation] = FinalScore
    }

    log_final_scores(final_scores)

    let selected_situation = get_situation_with_highest_total_score(final_scores) // max(final_score) with tiebreaker Timedelta
    intervention_selection(selected_situation)
    return null
}
```
### Algorithm 3: Select Intervention

The system should include a mechanism for selecting interventions. This should be triggered by Algorithm 2.

The system should select interventions based on their frequency and time delta (frequency: to avoid sending the same interventions to all couples too much; time delta: to avoid sending the same interventions many times in a row to the same couple). The total score for each intervention should be calculated as a combination of these two factors.

The selected intervention should be triggered with a 50% chance. The system should log the details of the intervention selection, including the current date and time, the selected situation, the selected intervention, and whether the intervention was triggered.

Flowchart Algorithm 3:  
<img src="https://github.com/Pascal-Kueng/AlgorithmV2/assets/108430531/5734945b-fcd6-4fcd-b652-5f7a181bf82b" width=450>


Pseudo code for Algorithm 3: Select Intervention:

```javascript
// no Cronjob, only triggered by Algorithm 2. 
function intervention_selection(selected_situation) {
    let intervention_pool = get_intervention_pool(selected_situation)
    
    let total_scores = {}
    foreach (intervention in intervention_pool) {
        let Frequency = get_frequency(intervention)
        let Timedelta = get_timedelta(intervention)
        let TotalScore = Frequency + Timedelta

        total_scores[intervention] = TotalScore
    }
    log_total_scores(total_scores)
    let selected_intervention = get_intervention_with_highest_total_score(total_scores) // max(total_score) with tiebreaker Timedelta
    
    // trigger intervention with random 50% chance
    let is_triggered
    if (random() < 0.5) {
        trigger_intervention(selected_intervention) // for correct target, same as pilot. 
        is_triggered = true
    } else {
        is_triggered = false
    }

    log_intervention_selection(current_datetime(), selected_situation, selected_intervention, is_triggered)
    return null
}
```

