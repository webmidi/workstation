# workstation

Web MIDI workstation

[Project status: Just an idea...]


## Objective

- A web-based MIDI workstation.

- Instruments can be loaded dynamically from the internet.


## Overview

1. Enter the Web MIDI workstation.

2. Web MIDI workstation boots and loads the modules (devices, such as synth, MIDI effects, MIDI adaptor/transmitter, etc) by including a `<script>` tag to load the instrument dynamically.

3. The module registers with Web MIDI workstation.

    ```js
    WebMIDIWorkstation.register({
      name: 'Piano synthesizer',
      inputMIDIPorts: 1,
      outputMIDIPorts: 0, // because it will use Web Audio API to emit sound.
      initialize: initializePianoSynthesizer
    })
    ```

4. The module shows up in the modules palette. When user selects it, the module will be initialized.

5. The workstation initializes the module by calling the initialization function. It will have this signature.

    ```js
    function initializePianoSynthesizer (options: ModuleInitializationOptions): ModuleInstance
    ```

    ### ModuleInitializationOptions
    
    This is an Object with information necessary to instantiate a module.

    - `element` An HTMLElement representing the instrument’s view.

      This element is focussable (has tabindex) and will be able to receive keyboard events. The element is automatically focused after the module is initialized.

    - `now` A Function that returns the current timestamp.

    - `input` An Observable&lt;InputEvent&gt; representing the input events to the module.

      An InputEvent is an Object with `type` property:

      - `type: 'midi'`
        - `port` A Number representing the MIDI port. Ports are zero indexed.
        - `data` A Uint8Array representing the incoming MIDI message.


    ### ModuleInstance

    This is an Object which will be used by the workstation to route MIDI to other modules.

    - `output` An Observable&lt;OutputEvent&gt; that contains the actions that should be performed by the workstation.

      It will be used under this contract:

      - The `subscribe()` method will be called when the module is activated (first at initialization, and when reactivated after it’s deactivated).

      - The `subscribe()` method should return a `Disposable` object with `dispose()` method (to signify unsubscription). Rx Observables do this already.

      - The subscription will be `dispose()`’d when the module is deactivated (when user disables it, or when user destroys the module).

      - There will be no concurrent subscriptions.

      An OutputEvent is an Object with `type` property:
      
      - `type: 'midi'`
        - `port` An optional Number representing the MIDI port. It is assumed to be 0 by default.
        - `data` A Uint8Array representing the incoming MIDI message.

      - `type: 'call'`
        - `fn` A Function to be invoked by the workstation.
        - `args` An Array representing arguments to be called.

      This allows your instrument to be side-effect free.


## Modules

- __MIDI Input__ routes input from Web MIDI API into the workstation.

- __MIDI Output__ routes MIDI events from the workstation to Web MIDI API.

- __Keyboard Input__ lets user emit MIDI note events with the keyboard.

  - On-screen keyboard should be provided. It should accept touch events. It can be maximized to provide full-screen 3-octave touch keyboard (e.g. for iPad).

  - Two octaves.
    - Upper: <kbd>Q</kbd><sup><kbd>2</kbd></sup><kbd>W</kbd><sup><kbd>3</kbd></sup><kbd>E</kbd><kbd>R</kbd><sup><kbd>5</kbd></sup><kbd>T</kbd><sup><kbd>6</kbd></sup><kbd>Y</kbd><sup><kbd>7</kbd></sup><kbd>U</kbd><kbd>I</kbd><sup><kbd>9</kbd></sup><kbd>O</kbd><sup><kbd>0</kbd></sup><kbd>P</kbd><kbd>[</kbd><sup><kbd>=</kbd></sup><kbd>]</kbd>
    - Lower: <kbd>Z</kbd><sup><kbd>S</kbd></sup><kbd>X</kbd><sup><kbd>D</kbd></sup><kbd>C</kbd><kbd>V</kbd><sup><kbd>G</kbd></sup><kbd>B</kbd><sup><kbd>H</kbd></sup><kbd>N</kbd><sup><kbd>J</kbd></sup><kbd>M</kbd><kbd>,</kbd><sup><kbd>L</kbd></sup><kbd>.</kbd><sup><kbd>;</kbd></sup><kbd>/</kbd>

  - Supports transposition:

    | Key | Transpose | Major key | Minor key |
    | --- | --------- | --------- | --------- |
    | <kbd>Esc</kbd> | -6 | F<sup>♯</sup> | E<sup>♭</sup>m |
    | <kbd>F1</kbd> | -5 | G | Em |
    | <kbd>F2</kbd> | -4 | A<sup>♭</sup> | Fm |
    | <kbd>F3</kbd> | -3 | A | F<sup>♯</sup>m |
    | <kbd>F4</kbd> | -2 | B<sup>♭</sup> | Gm |
    | <kbd>F5</kbd> | -1 | B | G<sup>♯</sup>m |
    | <kbd>F6</kbd> | ±0 | C | Am |
    | <kbd>F7</kbd> | +1 | C<sup>♯</sup> | B<sup>♭</sup>m |
    | <kbd>F8</kbd> | +2 | D | Bm |
    | <kbd>F9</kbd> | +3 | E<sup>♭</sup> | Cm |
    | <kbd>F10</kbd> | +4 | E | C<sup>♯</sup>m |
    | <kbd>F11</kbd> | +5 | F | Dm |
    | <kbd>F12</kbd> | +6 | F<sup>♯</sup> | E<sup>♭</sup>m |
    | <kbd>&larr;</kbd> | -=1 |
    | <kbd>&rarr;</kbd> | +=1 |

  - Supports octave shifting via arrow keys: <kbd>&uarr;</kbd>, <kbd>&darr;</kbd>

  - Supports note velocity by using Doppler.js to listen to finger’s speed.

  - Supports note velocity and aftertouch by scrolling mouse (using two fingers on trackpad while playing notes).

- WebMIDILink wrapper.
