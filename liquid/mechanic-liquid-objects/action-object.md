# Action object

 The action object describes an _action_ that was defined \(see [An introduction to actions](https://docs.usemechanic.com/article/415-an-introduction-to-actions)\), and the _run_ that was performed for that action \(see [An introduction to runs](https://docs.usemechanic.com/article/425-an-introduction-to-runs)\).

 As such, the action object _only_ comes into play with tasks that subscribe to mechanic/actions/perform, analyzing the results of action runs, for the purpose of performing followup work. Learn more about this technique: [Responding to action results](https://docs.usemechanic.com/article/431-responding-to-action-results).

##  How to access it

*  Use `{{ action }}` in tasks responding to mechanic/actions/perform

## What it contains

* `type` – a lowercase string, defining the [action type](https://docs.usemechanic.com/article/412-all-action-types)
* `options` – the options with which the action was configured
* `run` – an object containing the following attributes, describing the run that was generated by the action's performance
  * `id` – a string containing the UUID for this action
  * `ok` – a boolean, true for a successful action run, and false for a failure
  * `error` – either null, or a string containing the error message returned for a failed action
  * `result` – the data returned from the action; format varies by action type
  * `result_meta` – an object containing useful performance-related data
  * `attempts` – an integer, indicating the number of times the action was attempted before final success or failure
  * `enqueued_at` – the time at which this action was sent to our queue for execution
  * `started_at` – the time at which the action started
  * `stopped_at` – the time at which the action stopped
  * `elapsed_time_ms` – the number of milliseconds this action required, from start to stop
