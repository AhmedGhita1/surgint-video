# ORION

*Operating-Room Interval retrieval frOm Natural language*

natural language event retrieval over long, multi-camera operating room videos.

the systems indexes synchronized OR video streams and retrieves relevant temporal
segments from text queries such as:

- *"find whwen the surgeon is sawing"*
- *"show when the robot is being calibrated"*
- *"retrieve moments where instruments are being prepared"*

three levels of retrieval targets:
- workflow event/phase: *"find robot calibration*
- interaction: *"find when the surgeon manipulates the drill"*
- fine event: *"find hammering*

### data

the project uses the **MM-OR** dataset for development:

- 17 full procedures of ~90 minutes + 22 shorter clips
- 92,983 timepoints
- 25, 277 manually annotated timepoints

- 5 synchronized ceiling RGB-D cameras
- 3 higher-quality detail RGB cameras aimed at certain areas. to capture instruments, hand
    movements, and procedural cues.
- 1 low-exposure surgical-field camera

MM-OR provides semantic scene graphs describing entities and their relationships.

entities include:
    - `head surgeon`
    - `assistant surgeon`
    - `nurse`
    - `patient`
    - `robot`
    - `drill`
    - `hammer`
    - `saw`
    - `instrument table`
    - `operating table`

action predicates include:
    - `assisting`
    - `calibrating`
    - `cleaning`
    - `cutting`
    - `drilling`
    - `hammering`
    - `holding`
    - `manipulating`
    - `preparing`
    - `sawing`
    - `scanning`
    - `suturing`
    - `touching`

#### download

MM-OR is hosted by TUM CAMP. access requires filling the form linked from
[egeozsoy/MM-OR](https://github.com/egeozsoy/MM-OR) and agreeing to the terms of use.

```bash
cd data
./download_mm-or.sh --dry-run
./download_mm-or.sh
```

resumable — each archive gets a marker in `markers/`, so re-running skips whatever
already finished. archives are deleted after extraction.


## system story

### offline ingestion

`raw procedure` -> `synchronized video` -> `chunking` -> `GPU inference` -> `embeddings` -> `index`

### serving
`REST/Fast API` -> `query embedding` -> `vector search` -> `reranking` -> `clip extraction` -> `response`

### UI
a 90-minute OR timeline with retrievad events and synchronized camera views


### Deployment
`Docker` + `GPU inference` + `vector DB` + `API`


