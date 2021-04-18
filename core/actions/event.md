# Event

The **Event** action is for generating custom events in the [**User event domain**](). It's used to queue up follow-up work, either immediately or in the future, and can be useful when designing complex workloads, separating work between tasks.

Events generated by this action may be responded to by other tasks, or by the task that generated this action.

Events generated by this action are [**child events**](../events/parent-and-child-events.md) of the event responsible for the current action.

## Options

| Option | Description |
| :--- | :--- |
| `topic` | Required; a string specifying an [event topic](../events/topics.md) of the form "user/\*/\*" |
| `data` | Required; any JSON value \(including `null`\), to be used as the event data |
| `run_at` | Optional; a Unix timestamp integer, or any string that can be parsed as a time |
| `task_ids` | Optional, cannot be used with `task_id`; an array of task UUID strings, specifying which tasks are allowed to respond to this event |
| `task_id` | Optional, cannot be used with `task_ids`; a string containing a single task UUID, specifying which task is allowed to respond to this event |

### Notes

If a `run_at` value specifies a time in the past, the new event will be run immediately.

Tasks specified by `task_ids` or `task_id` must subscribe to the event topic being used. As with all subscriptions, [offsets](../tasks/subscriptions.md#offsets) may be used, and will be respected.

## Examples

### Using the Event tag

{% tabs %}
{% tab title="Liquid" %}
```javascript
{% assign data = hash %}
{% assign data["foo"] = "bar" %}

{% action "event", topic: "user/foo/bar", data: data, task_id: task.id %}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "action": {
    "type": "event",
    "options": {
      "topic": "user/foo/bar",
      "data": {
        "foo": "bar"
      },
      "task_id": "293b7040-6689-4eb1-8b5d-64f4d33eb2ae"
    }
  }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Liquid" %}
```javascript
{% assign one_day_in_seconds = 60 | times: 60 | times: 24 %}

{% action "event" %}
  {
    "topic": "user/foo/bar",
    "task_id": {{ task.id | json }},
    "run_at": {{ "now" | date: "%s" | plus: one_day_in_seconds | json }},
    "data": {
      "foo": "bar"
    }
  }
{% endaction %}
```
{% endtab %}

{% tab title="JSON" %}
```javascript
{
  "action": {
    "type": "event",
    "options": {
      "topic": "user/foo/bar",
      "task_id": "293b7040-6689-4eb1-8b5d-64f4d33eb2ae",
      "run_at": 1613158259,
      "data": {
        "foo": "bar"
      }
    }
  }
}
```
{% endtab %}
{% endtabs %}

### Scheduling future events

#### Using run\_at

This task emails a customer daily until their order is paid. It works by scheduling a follow-up run of the same task, one day in the future, using the `run_at` option.

{% tabs %}
{% tab title="Subscriptions" %}
```javascript
shopify/orders/create
user/orders/unpaid_reminder
```
{% endtab %}

{% tab title="Code" %}
```javascript
{% if event.preview %}
  {% assign order = hash %}
  {% assign order["id"] = 1234568790 %}
  {% assign order["name"] = "#1234" %}
{% elsif event.topic == "user/orders/unpaid_reminder" %}
  {% assign order = shop.orders[event.data.order_id] %}
{% endif %}

{% unless order.financial_status == "paid" %}
  {% action "email" %}
    {
      "to": {{ order.email | json }},
      "reply_to": {{ shop.customer_email | json }},
      "subject": "Order {{ order.name }} still needs to be paid",
      "body": "Please get in touch, stat!",
      "from_display_name": {{ shop.name | json }}
    }
  {% endaction %}

  {% assign one_day_in_seconds = 60 | times: 60 | times: 24 %}

  {% action "event" %}
    {
      "topic": "user/orders/unpaid_reminder",
      "task_id": {{ task.id | json }},
      "run_at": {{ "now" | date: "%s" | plus: one_day_in_seconds | json }},
      "data": {
        "order_id": {{ order.id | json }}
      }
    }
  {% endaction %}
{% endunless %}
```
{% endtab %}
{% endtabs %}

#### Using subscription offsets

This task emails a customer daily until their order is paid. It works by firing the follow-up event immediately, using a subscription offset to respond to it a day later.

{% tabs %}
{% tab title="Subscriptions" %}
```javascript
shopify/orders/create
user/orders/unpaid_reminder+1.day
```
{% endtab %}

{% tab title="Code" %}
```javascript
{% if event.preview %}
  {% assign order = hash %}
  {% assign order["id"] = 1234568790 %}
  {% assign order["name"] = "#1234" %}
{% elsif event.topic == "user/orders/unpaid_reminder" %}
  {% assign order = shop.orders[event.data.order_id] %}
{% endif %}

{% unless order.financial_status == "paid" %}
  {% action "email" %}
    {
      "to": {{ order.email | json }},
      "reply_to": {{ shop.customer_email | json }},
      "subject": "Order {{ order.name }} still needs to be paid",
      "body": "Please get in touch, stat!",
      "from_display_name": {{ shop.name | json }}
    }
  {% endaction %}

  {% assign one_day_in_seconds = 60 | times: 60 | times: 24 %}

  {% action "event" %}
    {
      "topic": "user/orders/unpaid_reminder",
      "task_id": {{ task.id | json }},
      "data": {
        "order_id": {{ order.id | json }}
      }
    }
  {% endaction %}
{% endunless %}
```
{% endtab %}
{% endtabs %}
