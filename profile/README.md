<img src="https://golang.design/favicon.svg" alt="logo" height="150" align="right" />

# The golang.design Initiative

We write Go for the parts the standard library leaves to you: the clipboard, the
GPU, the compiler, the thread you are pinned to. Everything imports under
`golang.design/x/...`, ships without a C toolchain wherever we can manage it, and
tells you up front what it will not do.

Building things in the open since September 2020.

## What we are building now

### accel: run compute on the GPU from Go, with `CGO_ENABLED=0`

You write the kernel in a subset of Go. `accel` compiles it ahead of time and
dispatches it to whatever the machine has, today the CPU or Metal, from a
cgo-free binary.

```go
//accel:kernel workgroup=64
func Scale(t accel.Thread, in []float32, out []float32) {
	i := t.GlobalID().X
	if i < uint32(len(out)) {
		out[i] = in[i] * 2
	}
}
```

`go generate` turns that into a compiled kernel record. Bindings, and whether
each one is read or written, are inferred from the body, so you never declare
them. The host side opens the best device available and falls back to the CPU
backend, running the same program either way.

```sh
go get golang.design/x/accel
go get -tool golang.design/x/accel/cmd/accel-kernel
```

The compute and tensor API is frozen and already carries an inference framework
on top of it. Graphics runs on both backends and is still settling. Vulkan,
D3D12, OpenGL and WebGPU are designed, not written.

**[Read the guide](https://golang.design/s/accel)** · [pkg.go.dev](https://pkg.go.dev/golang.design/x/accel)

### nanogo: a Go compiler you can read end to end

Source text to a `goobj` file that `go tool link` links against the real Go
runtime, under a stated budget of 40,000 lines. Point the go command at it for
the packages you name, and everything else still goes through `gc`:

```sh
go install golang.design/x/nanogo/cmd/nanogo@latest
echo main > allowlist
NANOGO_ALLOWLIST=./allowlist go build -toolexec=nanogo ./...
```

Set `NANOGO_LOG=./log` and it records what it compiled, what it handed back, and
what it turned down. It refuses closures, `append`, type assertions, `defer`,
maps and channels today, so most Go programs will not build. A refusal names the
function and the construct instead of emitting something wrong. The measure of
the project is compiling its own source to a fixed point.

**[Follow along](https://golang.design/s/nanogo)** · [pkg.go.dev](https://pkg.go.dev/golang.design/x/nanogo)

## Packages you can use today

| Import | What you get | Latest |
| --- | --- | --- |
| [`golang.design/x/clipboard`](https://golang.design/s/clipboard) | Read, write and watch the system clipboard: text, images, files and your own MIME types. macOS, Linux (X11 and Wayland), Windows, BSD, iOS, Android, js/wasm. Cgo-free on desktop. | `v0.9.0` |
| [`golang.design/x/hotkey`](https://golang.design/s/hotkey) | Register a system-wide shortcut and get an event when it fires, whether or not your window has focus. macOS, Linux (X11), Windows. | `v0.6.1` |
| [`golang.design/x/runtime`](https://golang.design/s/runtime) | The runtime knobs the standard library does not export: goroutine IDs, OS-thread caps, `mainthread` for the platform API that only accepts calls from the main thread, plus `thread` and `cgo`. | `v0.3.0` |
| [`golang.design/x/x11`](https://golang.design/s/x11) | Talk to an X server over its socket in pure Go. No cgo, no `libX11`, no display server assumptions. | `v0.2.0` |
| [`golang.design/x/chann`](https://golang.design/s/chann) | One generic channel type that is buffered, unbuffered or unbounded depending on how you construct it. Pick unbounded and a send never blocks. | `v0.2.1` |
| [`golang.design/x/lockfree`](https://golang.design/s/lockfree) | Concurrent data structures sorted by the guarantee they actually give you: import `lf` for lock-free, `wf` for wait-free. Each type documents its own progress bound. | `v0.1.0` |
| [`golang.design/x/reflect`](https://golang.design/s/reflect) | `DeepCopy[T any](src T) T`, which copies a value including unexported struct fields and hands back a fully independent one. The external implementation of proposal [go.dev/issue/51520](https://go.dev/issue/51520). | `v0.1.1` |

## Tools

- **[Go SSA Playground](https://golang.design/gossa)**: paste Go, read the SSA the compiler builds from it
- **[bench](https://golang.design/s/bench)**: `benchstat` and `perflock` behind one command, so a benchmark run is performance-locked and statistically analyzed without extra flags
- **[redir](https://golang.design/s/redir)**: the redirector serving every `golang.design/s/` link on this page
- **[code2img](https://golang.design/s/code2img)**: turn a snippet into an image through the carbon.now.sh API, with an iOS Shortcut included
- **[tgstore](https://golang.design/s/tgstore)**: encrypted object storage with no size limit, backed by Telegram

## Read

- **[Go: Under The Hood](https://golang.design/under-the-hood)**: how the runtime, compiler and toolchain work, with the source in front of you
- **[Go: Questions](https://github.com/golang-design/go-questions)**: Go 程序员面试笔试宝典, the language taught through the questions interviewers ask
- **[Go: A Documentary](https://golang.design/history)**: the talks, threads and design documents that got Go here
- **[golang.design/research](https://golang.design/research)**: shorter notes on whatever we just ran into

Older experiments stay up rather than getting deleted:
[go2generics](https://github.com/golang-design/go2generics) collected the Go 2
generics designs while they were still proposals.

## Contribute

Open an issue on the repository you are using. Bug reports that include the OS,
the Go version and a program we can run are the ones that get fixed first. If
you want to help build something instead, `accel` and `nanogo` are the two with
the most surface left to cover.

<img src="https://changkun.de/urlstat?mode=github&repo=golang-design/.github" alt="visitors" align="right" />
