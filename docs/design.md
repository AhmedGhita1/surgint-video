# ARGUS Design Guide

*Natural Language Event Retrieval for Surgical Procedures Recordings*

## A. purpose & use-case

Operating rooms can be instrumented with several synchronized cameras, producing recordings
that run for hours across simultaneous views. These recordings are the raw material for
surgical workflow analysis, training review, and retrospective study, yet they are consumed
linearly: locating a particular moment requires an expert to scrub through footage that is
overwhelmingly uneventful. Moreover, recordings accumulate faster than anyone can watch them.

ARGUS addresses this problem.


## B. problem statement

Given a set of synchronized video streams covering a single surgical procedure, together with a natural language description of an event, return a list of intervals on the procedure timeline at which the described event occurs, ranked by relevance, each with the streams ordered by
how likely they are to show it.

- A researcher opens a finished 90-minute knee replacement recorded by 3 cameras. 
- They are looking for every moment the saw was used. 
- Rather than scrubbing through 3 timelines, they type *"find when the surgeon is sawing"*.
- ARGUS answers with a handful of moments: 41:12, 43:50, 52:03.
- For each moment it orders the 3 views by how likely they are to show the event.
- They ask the next question: *"retrieve moments where instruments are being prepared"*.
- repeat


---

## C. system soncept

ARGUS is a retrieval system, it ranks time intervals in a recording by how well they match a
text query (event description), and the user decides which of them are relevant.

It runs in two phases:

1. **preparation.** Given the synchronized streams of one procedure, ARGUS does preparation
   work once, up front, afterwards it's ready for query.

2. **query.** Given one text query, ARGUS returns ranked time intervals on that procedure's
   timeline.

--- 
### preparation

preparation encodes a recording into a time-indexed representation that can be matched against text queries.

it has to do 3 things:
    1. **encode**: derive a representation from the recording, that is comparable with text.
    2. **align**: preserve the temporal origin of every element, so a match resolves to an interval on the procedure timeline and to a video stream.
    3. **persist**: store the representation in a structure that supports low-latency search.

**open decisions**:
*ordered by dependency*
1. scope
2. budget
3. representation
4. temporal resolution

representation >> resolution >> budget >> scope

### scope


### budget


### representation


### resolution


## D. requirements & constraints


## E. architecture
decomposition into subsystems and the contracts between them

## F. detailed design

