# time-tracker

## goals
  - exercise software engineering practices/learning
  - use the tool as a lab for my experimentation, AI stuff, A/B testing etc.
  - ...

# THOUGHTS / VISION

## use cases:
  * general:
    - prediction of what is going to be entered based on previous/historic interaction:
      - task duration/title/category/type based on day of the week and time of the day
    - agentic features like:
      - take actions based on entered data entered (either an embedded agent or a connection with a Personal Agent like Hermes Agent?)
      - generate insights of what a Personal Agent could be doing for that user (maybe just insights available for free users and the actions would be available for paid users)
      - ...
    - ability to express the emotions/feelings for each entry and an overall feeling BeginOD and EndOD
    - ability to register what I've learned for each entry and an overall EOD
    - ability to enter: WFH or Office (or leisure) (?)
    - sync data between mobile and desktop
  * desktop:
    - when opening the app, the main screen will always be the log screen (multiple lines) with the data entry at the bottom and previous logs above
  * mobile:
    - on the lock screen, have the ability to hold the icon and see what was the last/current entry and either close the current one or begin a new one
    - ...

## input:
  - user enters 'log entries' with:
    - `date` | `startTime` | `endTime` | `title` | `category` | `type/ticket` | `description` | `highlights`
      - `date` suggests the current date
      - `startTime` suggests the current time (rounded with 15min granularity)
      - `endTime` suggests the current time (rounded with 15min granularity) if `startTime` has been filled some time ago (more than 15min ago)
      - `title` | `category` | `type` are *predicted* based on the `startTime` and historical app usage/user behaviour (__AI__) (maybe just: tags?)
      - `description` and `highlights` are free form (not sure if it'd be possible to *predict*)
  - when typing new logs times (start/end), the app should:
    - identify possible overlaps with existing logs (for consistency)
    - (__IA__) predict the category/type of the entry based on the time of the day or the short description that the user is typing
  - the data entry layout:
    - mimics a command line filled with logs (older logs on top, newer ones at the bottom) (not so sure about this...)
    - the fields for entereing logs sit at the bottom, from left to right, aligning with the already entered log entries
  - past log entries should be slightly dimmed
  
  - current class levels are the *classifications* and *types*:
    - `category` - cQA | cDev | cMeeting | cDayOff | cMisc
    - `type` - ticket# (sometimes: local-llms | AIWG)
    - `highlights` - things that positively/negatively happened during that log entry (used for generating report with highlights)

## output (query/filtering/reporting):
  - label on the top of the main screen informing the amount of time worked that *day* and that *week*
  - *reports*:
    - (__IA__) one-to-one meetings focusing on achievements (from the highlights)
    - time distribution by categories per period (week/month/year) (`meetings` | `qa` | `dev` | `study` ...)
  - *filters*: (on the same main screen)
    - by any of the category/type by selecting one of them and choosing `filter`

## general:
  - (__IA__) interface to communicate with Hermes Agent. HA should be able to gather data to know what's happening with the user and feed periodic conversations

## planned approach
* v1: (just replicate what I have nowadays with Apple Numbers)
  - input: data entry with suggestions (not predictions yet) and immediate visualization (data above the entry fields)
  - query: range / category / type
* v2:
  - reports: one-to-one
  - predictions (__AI__)
  - MCP





## possible names:
  - *YATTA*: Yet Another Time Tracking Application
    - YATTA - time tracking on Toggl: https://apps.apple.com/ca/app/yatta-time-tracking-on-toggl/id793812713
    - https://dandys-world-robloxhorror.fandom.com/wiki/Yatta
    - https://yatta.ca
    - https://en.wikipedia.org/wiki/Yatta,_Hebron
    - Green Leaves "Yatta!" is a 2001 parody song by the fictional Japanese boy band Green Leaves. The song title, yatta, is the past tense of the Japanese verb yaru, an exclamation meaning "It's done!", "I did it!", "Ready!" or "All right!" The song and video have been used as a web culture in-joke on many different websites.
  - *NYATTA*: Not Yet Another Time Tracking Application
    - no apps on AppStore with that name (why?!)
    - Nyatta is a last name of some Oriental people
    - Sounds like Miatta (Mazda)
    - https://www.sullcrom.com/Lawyers/Inosi-M-Nyatta
    - https://womenworthwatching.com/inosi-m-nyatta/
    - https://www.salt.org/speakers/shu-nyatta
    - https://www.bloomberglinea.com/english/nyatta-says-new-growth-equity-fund-not-chasing-unicorns-eyeing-brazil-and-mexico/
    - https://www.nyattackvbclub.com
    - 

## possible domains:
  - yatta.github.com
  - yattapp.ca
  - yatta-app.ca

## interesting apps in that category:
  - https://apps.apple.com/ca/app/workhours-time-tracker/id1435481677: WorkHours: Time Tracker
  - https://apps.apple.com/ca/app/timesheet-track-hours/id6503605414: Timesheet: Track hours
  - https://apps.apple.com/ca/app/actitime-time-tracker/id6470903011: actiTIME Time Tracker
  - https://apps.apple.com/ca/app/hours-tracker-focus-keeper/id1438079949: Hours tracker & focus keeper
  - 