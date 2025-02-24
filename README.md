# ipfs-car 🚘✨⬢

> Convert files to content-addressable archives (.car) and back

## Description

`ipfs-car` is a library and CLI tool to pack & unpack files from [Content Addressable aRchives (CAR)](https://github.com/ipld/specs/blob/master/block-layer/content-addressable-archives.md) file. A thin wrapper over [@ipld/car](https://github.com/ipld/js-car) and [unix-fs](https://github.com/ipfs/js-ipfs-unixfs).

Content-addressable archives store data as blocks (a sequence of bytes) each prefixed with the [Content ID (CID)](https://docs.ipfs.io/concepts/content-addressing/) derived from the hash of the data; typically in a file with a `.car` extension.

Use `ipfs-car` to pack your files into a .car; a portable, verifiable, IPFS compatible archive.

```sh
$ ipfs-car --pack path/to/files --output my-files.car
```

or unpack files from a .car, and verify that every block matches it's CID

```sh
$ ipfs-car --unpack my-files.car --output path/to/write/to
```

## Install

```sh
# install it as a dependency
$ npm i ipfs-car

# or use the cli without installing via `npx`
$ npx ipfs-car --help
```

## Usage

`--pack` files into a .car

```sh
# write a content addressed archive to the current working dir.
$ ipfs-car --pack path/to/file/or/dir

# specify the car file name.
$ ipfs-car --pack path/to/files --output path/to/write/a.car
```

`--unpack` files from a .car

```sh
# unpack files to a specific path.
$ ipfs-car --unpack path/to/my.car --output /path/to/unpack/files/to

# unpack specific roots
$ ipfs-car --unpack path/to/my.car --root <cid1> [--root <cid2>]

# unpack files from a .car on stdin.
$ cat path/to/my.car | ipfs-car --unpack
```

List the contents of a .car

```sh
# list the files.
$ ipfs-car --list path/to/my.ca

# list the cid roots.
$ ipfs-car --list-roots path/to/my.ca

# list the cids for all the blocks.
$ ipfs-car --list-cids path/to/my.ca
```

## API

To pack files into content-addressable archives, you can use the functions provided in:
- `ipfs-car/pack` for consuming a [CAR writer](https://github.com/ipld/js-car#carwriter) async iterable
- `ipfs-car/pack/blob` for getting a blob with the CAR file (**browser only**)
- `ipfs-car/pack/fs` for storing in the local file system (**Node.js only**)
- `ipfs-car/pack/stream` for writing to a writable stream (**Node.js only**)

⚠️ While packing files into CAR files, a [Blockstore](./src/blockstore) is used for temporary storage. All pack functions provide a default, but you can use other options (if supported on your runtime).

To unpack content-addressable archives to files, you can use the functions provided in:

- `ipfs-car/unpack` for getting an async iterable of the UnixFS entries stored in the CAR file
- `ipfs-car/unpack/fs` for writing the unpacked files to disk (**Node.js only**)

### `ipfs-car/pack`

Takes an [ImportCandidateStream](https://github.com/ipfs/js-ipfs/blob/master/packages/ipfs-core-types/src/utils.d.ts#L27) and returns a a [CAR writer](https://github.com/ipld/js-car#carwriter) async iterable.

```js
import { pack } from 'ipfs-car/pack'
import { MemoryBlockStore } from 'ipfs-car/blockstore/memory' // You can also use the `level-blockstore` module

const { root, out } = await pack({
  input: [new Uint8Array([21, 31, 41])],
  blockstore: new MemoryBlockStore()
})

const carParts = []
for await (const part of out) {
  carParts.push(part)
}
```

### `ipfs-car/pack/blob`

Takes an [ImportCandidateStream](https://github.com/ipfs/js-ipfs/blob/master/packages/ipfs-core-types/src/utils.d.ts#L27) and writes it to a Blob (**Browser only**).

```js
import { packToBlob } from 'ipfs-car/pack/blob'
import { MemoryBlockStore } from 'ipfs-car/blockstore/memory' // You can also use the `level-blockstore` module

const { root, car } = await packToBlob({
  input: [new Uint8Array([21, 31, 41])],
  blockstore: new MemoryBlockStore()
})
```

### `ipfs-car/pack/fs`

Takes a path on disk and writes it to CAR file (**Node.js only**).

```js
import { packToFs } from 'ipfs-car/pack/fs'
import { FsBlockStore } from 'ipfs-car/blockstore/fs'

await packToFs({
  input: `${process.cwd()}/path/to/files`,
  output: `${process.cwd()}/output.car`,
  blockstore: new FsBlockStore()
})
// output.car file now exists in process.cwd()
```

### `ipfs-car/pack/stream`

Takes a writable stream and pipes the CAR Writer stream to it (**Node.js only**).

```js
import fs from 'fs'
import { packToStream } from 'ipfs-car/pack/stream'
import { FsBlockStore } from 'ipfs-car/blockstore/fs'

const writable = fs.createWriteStream(`${process.cwd()}/output.car`)

await packToStream({
  input: `${process.cwd()}/path/to/files`,
  writable,
  blockstore: new FsBlockStore()
})
// output.car file now exists in process.cwd()
```

### `ipfs-car/unpack`

Takes a CAR reader and yields files to be consumed.

```js
import fs from 'fs'
import { unpack } from 'ipfs-car/unpack'

const inStream = fs.createReadStream(`${process.cwd()}/output.car`)
const carReader = await CarReader.fromIterable(inStream)

const files = []
for await (const file of unpack(carReader)) {
  // Iterate over files
}
```

### `ipfs-car/unpack/fs`

Takes a path to a CAR file on disk and unpacks it to a given path

```js
import { unpackToFs } from 'ipfs-car/unpack/fs'

await unpackToFs({
  input: `${process.cwd()}/my.car`,
  output: `${process.cwd()}/foo`
})
// foo now exists in process.cwd()
// it is either a file or a directory depending on the contents of the .car
```

Takes a stream to a CAR file and unpacks it to a given path on disc

```js
import fs from 'fs'
import { unpackStreamToFs } from 'ipfs-car/unpack/fs'

const input = fs.createReadStream(`${process.cwd()}/my.car`)

await unpackStreamToFs({
  input,
  output: `${process.cwd()}/foo`
})
// foo now exists in process.cwd()
// it is either a file or a directory depending on the contents of the .car
```
