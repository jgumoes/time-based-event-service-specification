# Create Time-Based Event


## Packet Structure
```
{
  (optional) E2E-CRC
  uint8_t eventID;
  uint8_t actionID;
  uint8_t parameters;
  sint32_t eventStart;
  uint16_t payload
  (optional) sint32_t eventEnd;
  (optional) uint8_t previousEventId
  (optional) uint16_t missedEventTimeout
}
```

## eventID

Must be unique across the application, and must be non-zero.

## actionID

Points to the action associated with the event. 

### reserved IDs

some actionIDs are reserved for specific events

| value | event | payload |
|-------|-------|---------|
| 253 | cancel cascadingEvent | 
| 254 | set DST value | value<sup>1</sup> |

1. DST value as per Device Time Service BLE specifications

## parameters

| bit | detail    |
|-----|-----------|
| 0   | Repeating Event |
| 1   | User Interaction Required |
| 2   | User Defined Data Included |
| 3   | Block Time Adjust |
| 4   |  |
| 5   |  |
| 6   |  |
| 7   |  |

**Repeating Event Bit**: if true, the event will be re-instantiated after it expires. If false, the event will trigger on the next available day, and then should be deleted from the application device's filesystem after expiry.

**User Interaction Required Bit**: If high, the event requires a user to interact with the device for the next event to be set. Only required if another time-based event requires 

**User Defined Data Included bit**: set high if the optional payload is included

**Block Time Adjust bit**: set high if time adjustments should be blocked. see **Missed Events** in the main readme for more information.
<!-- 
**Event Value**:

| value | event | payload |
|-------|-------|---------|
| 1     | trigger action | action ID / value |
| 2     | toggle action<sup>1</sup> | action ID / value |
| 3     | increasing 

1 eventEnd must be given, the payload should be reverted  -->

## eventStart

The timestamp for the start of the event is in BCD. It is the format that most RTC ICs expect for on-chip alarms, and it doesn't actually take more space than a seconds timestamp (there are 86,400 seconds in a day, which is a 17 bit number so will require 4 bytes).

| bit | value    |
|-----|-----------|
| 0-3  | seconds (1s)  |
| 4-6  | seconds (10s)  |
| 7-10 | minutes (1s) |
| 11-13 | minutes (10s)|
| 14-17 | hours (1s) |
| 18-19 | hours (10s) |
| 20-29 | Type-Specific Operand |
| 30-31 | Time-Event type   |

note: if the timestamp is relative to midnight, it cannot exceed 86,399 seconds.

### Type-Specific Operand

**Day Of Week**
| bit | value    |
|-----|-----------|
| 20   | event triggers on Monday    |
| 21   | event triggers on Tuesday  |
| 22   | event triggers on Wednesday    |
| 23   | event triggers on Thursday  |
| 24   | event triggers on Friday |
| 25   | event triggers on Saturday   |
| 26   | event triggers on Sunday    |
| 27-29| unused |

**Date Of Month**
| bit | value    |
|-----|-----------|
| 20-23 | date (1s) |
| 24-25 | date (10s) |
| 26-29 | month in regular binary format |

The month won't fit with BCD, but RTC chipsets with alarm tend not to have a month option so this should not cause an inconvenience.

**Cascading**
| bit | value    |
|-----|-----------|
| 20-22 | days |
| 23-26 | weeks (1s) |
| 27-28 | weeks (10s) |
| 29 | Self-Triggering Event |

*Self-Triggering Event Bit*: If high, the event will be triggered by itself and will keep repeating until cancelled by the user. The server should return an error if User Interaction Bit is low.

### Time-Event type bit

| 31, 30 | event type |
| ------ | ---------- |
| 0, 1 | Day Of Week event |
| 1, 0 | Date Of Month event |
| 1, 1 | Cascading event |

## missedEventTimeout

If the event isn't triggered in time (i.e. because of a power outage, OS rebooted, etc.), how late can it still trigger? e.g. if an event is set at 5 pm, and missedEventTimeout is set for 5 minutes, the event can be triggered at any time between 5:00 pm and 5:05 pm. It will be triggered as early as possible. Defaults to 30 seconds. 