---
title: Count Days Without Meetings
date: 2025-03-24
lastmod: 2025-03-24
author:
  - Jimmy Lin
tags:
  - greedy
  - interval
  - line_sweep
draft: false
---

## Thinking Process

會用到 [[Merge Intervals]]。
## Code

Time Complexity: $O(n \log n)$, Space Complexity: $O(n)$

```cpp
class Solution {
public:
    int countDays(int days, const vector<vector<int>>& meetings) {
        if (meetings.empty()) return days;

        vector<vector<int>> merged = mergeMeetings(meetings);
        return countAvailableDays(days, merged);
    }

    vector<vector<int>> mergeMeetings(const vector<vector<int>>& meetings) {
        vector<vector<int>> sortedMeetings = meetings;
        sort(sortedMeetings.begin(), sortedMeetings.end());

        vector<vector<int>> merged;
        merged.push_back(sortedMeetings[0]);

        for (int i = 1; i < sortedMeetings.size(); ++i) {
            const auto& meeting = sortedMeetings[i];
            if (meeting[0] > merged.back()[1]) {
                merged.push_back(meeting);
            } else {
                merged.back()[1] = max(merged.back()[1], meeting[1]);
            }
        }
        return merged;
    }

    int countAvailableDays(int d, const vector<vector<int>>& meetings) {
        int available = 0;

        // Before first meeting
        available += meetings[0][0] - 1;

        // Gaps between merged meetings
        for (int i = 1; i < meetings.size(); i++) {
            available += (meetings[i][0] - meetings[i - 1][1] - 1);
        }

        // After last meeting
        available += d - meetings.back()[1];

        return available;
    }
};
```

也可以使用 [[Line Sweep Problems]] 的技巧來解。

```python
class Solution:
    def countDays(self, days: int, meetings: List[List[int]]) -> int:
        events = []
        for start, end in meetings:
            events.append([start, 1])
            events.append([end + 1, -1])

        events.sort()

        active_meetings = 0
        days_available_for_work = 0
        last_meeting_day = 1
        for day, change in events:
            if active_meetings == 0 and day > last_meeting_day:
                days_available_for_work += day - last_meeting_day

            active_meetings += change
            last_meeting_day = day
        
        if days >= last_meeting_day:
            days_available_for_work += days - last_meeting_day + 1
        
        return days_available_for_work



```
## Source
- [Count Days Without Meetings](https://leetcode.com/problems/count-days-without-meetings/)