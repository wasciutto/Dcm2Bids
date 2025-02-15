# How to create a configuration file

## Configuration file example

```json
{
  "descriptions": [
    {
      "dataType": "anat",
      "modalityLabel": "T2w",
      "criteria": {
        "SeriesDescription": "*T2*",
        "EchoTime": 0.1
      },
      "sidecarChanges": {
        "ProtocolName": "T2"
      }
    },
    {
      "dataType": "func",
      "modalityLabel": "bold",
      "customLabels": "task-rest",
      "criteria": {
        "ProtocolName": "func_task-*",
        "ImageType": ["ORIG*", "PRIMARY", "M", "MB", "ND", "MOSAIC"]
      }
    },
    {
      "dataType": "fmap",
      "modalityLabel": "fmap",
      "intendedFor": 1,
      "criteria": {
        "ProtocolName": "*field_mapping*"
      }
    },
    {
      "dataType": "func",
      "modalityLabel": "bold",
      "customLabels": "task-learning",
      "criteria": {
        "SeriesDescription": "bold_task-learning"
      },
      "sidecarChanges": {
        "TaskName": "learning"
      }
    },
    {
      "dataType": "fmap",
      "modalityLabel": "epi",
      "criteria": {
        "SeriesDescription": "fmap_task-learning"
      },
      "IntendedFor": 2,
      "sidecarChanges": {
        "TaskName": "learning"
      }
    }
  ]
}
```

The `descriptions` field is a list of descriptions, each describing some
acquisition. In this example, the configuration describes five acquisitions, a
T2-weighted, a resting-state fMRI, a fieldmap, and an fMRI learning task with
another fieldmap.

Each description tells dcm2bids how to group a set of acquisitions and how to
label them. In this config file, Dcm2Bids is being told to collect files
containing

```json
{
  "SeriesDescription": "AXIAL_T2_SPACE",
  "EchoTime": 0.1
}
```

in their sidecars[^1] and label them as `anat`, `T2w` type images.

## criteria

dcm2bids will try to match the sidecars[^1] of dcm2niix to the descriptions of
the configuration file. The values you enter inside the criteria dictionary are
patterns that will be compared to the corresponding key of the sidecar.

The pattern matching is shell-style. It's possible to use wildcard `*`, single
character `?` etc ... Please have a look at the [GNU documentation][gnu-pattern]
to know more.

For example, in the second description, the pattern `*T2*` will be compared to
the value of `SeriesDescription` of a sidecar. `AXIAL_T2_SPACE` will be a match,
`AXIAL_T1` won't.

`dcm2bids` has a `SidecarFilename` key, as in the first description, if you
prefer to also match with the filename of the sidecar. Note that filename are
subject to change depending on the dcm2niix version in use.

You can enter several criteria. **All criteria must match** for a description to
be linked to a sidecar.

## dataType

It is a mandatory field. Here is a definition from `bids v1.2.0` :

> Data type - a functional group of different types of data. In BIDS we define
> six data types: func (task based and resting state functional MRI), dwi
> (diffusion weighted imaging), fmap (field inhomogeneity mapping data such as
> field maps), anat (structural imaging such as T1, T2, etc.), meg
> (magnetoencephalography), beh (behavioral).

## modalityLabel

It is a mandatory field. It describes the modality of the acquisition like
`T1w`, `T2w` or `dwi`, `bold`.

## customLabels

It is an optional field. For some acquisitions, you need to add information in
the file name. For resting state fMRI, it is usually `task-rest`.

To know more on how to set these fields, read the [BIDS
specifications][bids-spec].

For a longer example of a Dcm2Bids config json, see
[here](https://github.com/unfmontreal/Dcm2Bids/blob/master/example/config.json).

Note that the different bids labels must come in a very specific order to be bids valid filenames. 
If the customLabels fields that are entered that are in the wrong order,
then dcm2bids will reorder them for you.

For example if you entered:

```json
"customLabels": "run-01_task-rest"
```

when running dcm2bids, you will get the following warning:

```bash
WARNING:dcm2bids.structure:✅ Filename was reordered according to BIDS entity table order:
                from:   sub-ID01_run-01_task-rest_bold
                to:     sub-ID01_task-rest_run-01_bold
```

## sidecarChanges

Optional field to change or add information in a sidecar.

## intendedFor

Optional field to add an `IntendedFor` entry in the sidecar of a fieldmap. Just
put the index or a list of indices of the description(s) that's intended for.

Python index begins at `0` so in the example, **`1`** means it is intended for
`task-rest_bold` and **`2`** is intended for `task-learning` which will be
renamed to only `learning` because of the
`"sidecarChanges": { "TaskName": "learning" }` field.

## Multiple config files

It is possible to create multiple config files and iterate the `dcm2bids`
command over the different config files to structure data that have different
parameters in their sidecar files.

[^1]:
    For each acquisition, `dcm2niix` creates an associated `.json` file,
    containing information from the dicom header. These are known as
    **sidecars**. These are the sidecars that `dcm2bids` uses to filter the
    groups of acquisitions.

    To define the filters you need, you will probably have to review these
    sidecars. You can generate all the sidecars for an individual participant
    using the [dcm2bids_helper](./use-main-commands.md#tools) command.

[bids-spec]: https://bids-specification.readthedocs.io/en/stable/
[gnu-pattern]:
  https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
