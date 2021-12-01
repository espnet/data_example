# DATA EXAMPLE

An exmpale of the data structure required by ESPnet. The required format can be found in `data`

```
|-- data
|   |-- test
|   |   |-- spk2utt  # Mapping a speaker-ID to a list of utterance-IDs
|   |   |-- text     # Mapping a utterance-ID to a text
|   |   |-- utt2spk  # Mappinng a utterance-ID to a speaker-ID
|   |   `-- wav.scp  # Mappinng a utterance-ID to a path of audio file
|   |-- train
|   |   |-- spk2utt
|   |   |-- text
|   |   |-- utt2spk
|   |   `-- wav.scp
|   `-- valid
|       |-- spk2utt
|       |-- text
|       |-- utt2spk
|       `-- wav.scp
```

## Concept

ESPnet follows the data strcutre developed by [Kaldi-asr](https://github.com/kaldi-asr/kaldi): A data-directory must contain some texts, `wav.scp`, `text`, and etc. which have common format to describe DNN corpus. The format is `space` separated and must be two columns. The first column is taken as `ID` and the second is some value.


```
<ID> <value>
<ID> <value>
...
```

e.g. `wav.scp` shows `<Sample ID> <Wave file path>`.



## Notes

- The directory name in `data` is arbitrary: `train`, `valid`, and `test` can be `tr`, `cv`, and `eval` for example.
- The path for `wav.scp` can be both absolute path or relative path from the base directory ( `egs2/<corpus-name>/<task-name>`, e.g. `egs2/an4/asr1`).
    ```
    uttidA /absolute/path/uttidA.wav
    uttidB ./relative/path/uttidB.wav
    ```
  - Maybe, we assume `monaural` and `16bit-signed-integer-pcm` audio file. Any sampling rates are okay. 
    -  Please check your audio format in advance using `sox` or `ffmpeg` for example.
        ```
        soxi youraudio.wav
        ```
  - (THIS IS EXTENSION BY ESPNET, NOT KALDI FORMAT) Not only `wav`, but also `flac` can be used (Supported audio format can be extended)
      ```
      uttidA a.flac
      uttidB b.flac
      ``` 
  - `wavs` directory exists at the same level of `data` in this example, but this is not requirement.
- If you don't have speaker information, it can be dummy data because actually most recipes don't use speaker information.
    ```
    uttidA dummy
    uttidB dummy
    uttidC dummy
    ...
    ```
- `spk2utt` can be gernated from `utt2spk` using [utils/utt2spk_to_spk2utt.pl](https://github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/utils/utt2spk_to_spk2utt.pl), and `utt2spk` can be gernated from from `spk2utt` using [utils/spk2utt_to_utt2spk.pl](https://github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/utils/spk2utt_to_utt2spk.pl).

    ```sh
    utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
    utils/spk2utt_to_utt2spk.pl data/train/spk2utt > data/train/utt2spk
    ```
- To check and force your directory to satisfy expected format, use [utils/validate_data_dir.sh](https://github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/utils/validate_data_dir.sh) and [utils/fix_data_dir.sh](https://github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/utils/fix_data_dir.sh)
    ```
    # check
    utils/validate_data_dir.sh --no-feats data/train
    # Force format (This is irreversible change)
    utils/fix_data_dir.sh data/train
    ```
- It's okay to contains the other files (`foo.txt` in the following example) in the data directory. They are not referred. 
    ```
    data
      `-- test
        |-- foo.txt
        |-- spk2utt
        |-- text 
        |-- utt2spk
        `-- wav.scp
    ```
    
## [Option] `segments`: Chuking long audio files into short segments

```
|-- segments
|-- spk2utt
|-- text
|-- utt2spk
`-- wav.scp
```

If your audio data is long recording and each audio file includes multiple utterances, you need to put `segments` file to specify the start time and end time of each utterance. The format is `<utterance_id> <wav_id> <start_time> <end_time>` (in seconds).

```
sw02001-A_000098-001156 sw02001-A 0.98 11.56
...
```
    
Note that if using `segments`, `wav.scp` has `<wav_id>` instead of `utterance_id`.

```
sw02001-A /path/to/sw02001-A.wav
...
```

## [Option] Change format using the pipeline mechanism of Unix system

`wav.scp` provides a feature to describe the file format of wavfiles converted by an arbitral command **without saving the files actually**. The usage is as following:

```
foo_id some_command /path/to/foo.wav |
```

Note that in our style, the environment variable for scripts are set by `path.sh`, so please check that the command exists in the `${PATH}`.


If a line ends with `|`, it indicates using this pipeline mechanism and our Python script derives the output data from the command via pipeline (We are using https://github.com/nttcslab-sp/kaldiio).

```
e.g. Change sampling rate, encoding, bits, etc.
sox input.wav -b16 -e unsigned-integer -r 16000 -t wav - |

e.g. Channel selection
sox stereo.wav -c 1 -t wav - |
sox stereo.wav -c 2 -t wav - |
```


## [Tips] `utils/fix_data_dir.sh`: Create a subset from selected data

## [Tips] `utils/combine_data.sh`: Combine multiple datadirs into a datadir
