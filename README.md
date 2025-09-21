//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

karma~ 2.0 research ideas

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

# Core Features:
- dynamic length
- varispeed (while recording) (up AND down) (better than linear interpolation?) (rubberband interpolation?)
- overdub/replace (does it fadein/doublebuffer?)
- fancy playback interpolation (especially at near zero playback speeds)
- position/window paradigm (normalized or other units?)
- jumping (normalized or other units) (ability to jump to markers instead?) (automatically set markers at beat divisions)
- declicking (fade or swanramp option?)
- smoother loop end boundary (double-buffer crossfade at loop end?)
- looping optional (one-shot)
- reverse/jump settable while recording
- signal-rate sync
- signal-rate transport/control
- lock to global transport

# New Features:
- timestretch (rubberband) (warp markers)
- decontruct object (record, interpolate, transport)
- varispeed option for playback *only* (e.g. recording with speed at 0.5, you will hear audio at 0.5 on playback)
- multiple transport/state options (3 button (start/play/record), 2 button ([record/overdub], [play/stop], 1 button (RC-1))
- delta mode playback (for scratch playback) (with extra fancy sinc interpolation?) + karma_play~
- multiply loop length (coupled with `set` message)
- lockout for “playhead dragging” (e.g. playhead touches loop point) (to avoid zippering)
- add abstractions like wrapping tape sample start/stop

# Spicier Features:
- adding effects to feedback path needs to be internal (filters, bitcrush, modulation) (CB Blooper has `Stability` parameter)
- timestretch tempo adapting
- automatic transient detection (on message), builds warp/transient markers that can be adjusted or jumped to
- insert mode for replace (adds in time?)
- append (use setloop and “jumptoempty”, “continueinitialloop”)
- undo layers (creating bigger-than-defined internal buffer)
- have a `gen~` / `rnbo~` version
- apply amplitude window to loop
- separate play/record heads? (simultaneous play and record?)
- define multiple loop points (play from 0.25 -> 0.5 and then 0.75 -> 0.85)
- define amount of loops (for global loop or individual sub-slices)
- threshold-based recording (sending `record` only puts `karma~` into an "armed" state, it won't start recording until a threshold is exceeded)

# Research
- look up undo layers in sooperlooper line6
- state machine funny business (define logic states) (including insert/append-y stuff) (including sample accurate considerations)

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

transport ideas

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

# One Button (RC-1)
- Hit BUTTON to begin recording a new loop (put into `initial loop` state)
- If in `initial loop` state, hitting BUTTON will being playing back loop (put into `play` state)
- If in `play` state, hitting BUTTON will go into `overdub` state (put into overdub state)
- If in either `play` or `overdub` state, a double-tap on BUTTON will go into a `stop` state
- If in `stop` state, hitting BUTTON will go into `play` state
- In any state, a press+hold on BUTTON will clear the current recording
- (once a loop has been created, the only way to start a new loop is to manually clear the previous one)
- (after a loop is recorded, the default state goes back and forth between `play` and `overdub`)


# Two Button ([record/overdub], [play/stop])
- Hit BUTT1 to begin recording a new loop (put into `initial loop` state)
- If you end the loop by hitting BUTT1, the loop will begin playing in `overdub` state
- If you end the loop by hitting BUTT2, the loop will go into `play` state

- If in `initial loop` state, hitting BUTT1 will go into `overdub` state
- If in `initial loop` state, hitting BUTT2 will go into `play` state

- If in `overdub` state, hitting BUTT1 will go into `play` state
- If in `overdub` state, hitting BUTT2 will go into `play` state

- If in `play` state, hitting BUTT1 will go into `overdub` state
- If in `play` state, hitting BUTT2 will go into `stop` state

- If in `stop` state, hitting BUTT1 will go into `intial loop` state
- If in `stop` state, hitting BUTT2 will go into `play` state


# Three Button (start/play/record)
- Hit RECORD to begin recording a new loop.
- If you end the loop by hitting RECORD, the loop will begin playing in OVERDUB mode.
- If you end the loop by hitting PLAY, the loop will begin playing.
- If you end the loop by hitting STOP, you define the loop length and stop playback.

- Hit PLAY to begin playing the newly created loop.
- If you hit RECORD, you will go into OVERDUB mode.
- If you hit PLAY, you will begin playback from the start of the defined position/window..
- If you hit STOP, you will stop playback.

- Hit RECORD while playing loop to enter OVERDUB mode.
- If you hit RECORD, you will go into PLAY mode.
- If you hit PLAY, you will go into PLAY mode.
- If you hit STOP, you will stop playback.


# Thoughts
- vanilla karma should have `record`, `play`, `stop` control the state machine
- have separate inputs for other functions (BUTT1 = record/overdub, BUTT2 = play/stop)
- separate butt (BUTT3?) for “one button” transport?
- overall state machien can only have a fininte amount of states, so control paradigms can be intertwined
- separate messages for `clear`
- “one button” input can take bangs (advance) or boolean input for press+hold or double-tap?

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

- Boss RC-5 only has 1 layer of undo
- UndoLooper has infinite and features for “back 5” and “bring it back” to recall many (https://undolooper.itch.io/undolooper)
- ChaseBliss Blooper has 8 layers of undo (https://www.chasebliss.com/blooper). all new layers are added onto the 8th layer
- EDP has no fixed limit but is based on memory (“the exact number of layers is not fixed and depends on available memory, which is influenced by your use of multiple loops and other function”)
- Valeton NUX MG-300 has one layer of undo

# Important Notes
- in Blooper a “layer” is anything added while recording/overdubing *until* play is hit (so many passes may be one “layer”)
- on EDP, `undo` also cancels most recent function (e.g. `record` -> `undo` comes out of record mode)
- many loopers also have a `redo` function

# Special Cases
- `insert` and `replace` need special consideration with layering
- insert mode would add time to all layers (adding silence to earlier layers)
- undoing an insert would revert the inserted audio (this is what the EDP does)
- what should replace mode do on an additional layer do to the underlying layer(s)?
