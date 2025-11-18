# linx

## API Spec

```coffee
var (
	...  # app state
)

onInstall => {  # when install this app
	...
	# should return initial state of this app
}

onStart => {  # when program start
	...
}

onState => {  # query app state
	...
}

onCmd Type, args => {   # tool definition
	# type(args) is ToolParams[Type]
	...
}

onVoice Text, v => {  # v.Data is converted to text
	# type(v) is VoiceInfo[Text]
	echo v.Who, v.Data, v.Language # v.Who say sth. in v.Language
	...
}

onVoice Stream, v => {  # v.Data is a stream, in signle/multiple channels
	# type(v) is VoiceInfo[Stream]
	...
}

type ToolParams[Type] struct {
	VoiceInfo VoiceInfo[Text]
	Type
}

type VoiceInfo[Type] struct {
	Who      string  # Voiceprint ID
	Language string  # Detected language of the voice
	Data     Type
}
```

## User Story

### Meeting Minutes

```coffee
type Start struct {
	_ "Start recording meeting minutes"
}

type Stop struct {
	_ "Stop recording meeting minutes"
}

var (
	started bool
)

onInstall => {
	accept Start
}

onCmd Start, args => {
	started = true
	accept Voice, Stop
}

onCmd Stop, args => {
	started = false
	accept Start
}

onVoice Stream, v => {
	# save v.Data
}
```

### Voice Translator

```coffee
type Start struct {
	_ "Start translating a conversation"
	TargetLanguage string "The target language to be translated. The default is English."
}

type Stop struct {
	_ "Stop translating current conversation"
}

var (
	started bool
	langs   [2]string
)

onInstall => {
	accept Start
}

onCmd Start, args => {
	started = true
	langs[0] = args.VoiceInfo.Language
	langs[1] = args.TargetLanguage
	accept Voice, Stop
}

onCmd Stop, args => {
	started = false
	accept Start
}

onVoice Text, v => {
	toLang := langs[1]
	if v.Language == toLang {
		toLang = langs[0]
	}
	text := translate(v.Data, toLang)
	speak text, v.Who  # speak in v.Who's voice
}
```
