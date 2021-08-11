---
title: ADD vs COPY in Dockerfile
date: 2021-08-09 22:06:48
tags: [Docker]
categories: DevOps
---

## TL;DR

忘了 `ADD`，使用 `COPY` 吧。

根據官方的 [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy)，`ADD` 和 `COPY` 基本上一樣，除了 `ADD` 提供了一些黑魔法，但基本上這些黑魔法都有其他的方法可以替代，而且 `ADD` 的這些額外的功能會造成執行結果難以直觀的預期，因此官方建議不要用 `ADD`。

## `ADD` and `COPY`

語法

```Dockerfile
COPY [--chown=<user>:<group>] <src> <dest>
ADD  [--chown=<user>:<group>] <src> <dest>
```

`ADD` 跟 `COPY` 基本的功能就是將檔案從 host 複製到 container 中，這個部分的功能是完全一樣的，並且支援 Go 的 [filepath.Match](https://pkg.go.dev/path/filepath#Match) 的路徑語法，幾個常用的例子例如

### `*` - 配對任意數量的任意字元

```Dockerfile
COPY hom* /mydir/
```

配對所有 `hom` 開頭的檔案如 `home.txt` `homogeneous.zip`

### `?` - 配對單一任意字元

```Dockerfile
COPY hom?.txt /midir/
```

配對如 `home.txt` `homy.txt`

## `ADD` 黑魔法

`ADD` 主要多了兩個功能：遠端下載、自動解壓縮

### 遠端下載

`ADD <src> <dest>` 中如果 `<src>` 是個遠端網址的話 `ADD` 指令會自動幫你下載檔案，例如

```Dockerfile
ADD https://example.com/big.tar.xz /usr/src/things/
```

### 自動解壓

如果 `<src>` 是個**本地**的 tar 壓縮檔案的話，`ADD` 會自動幫你解壓縮，例如

```Dockerfile
ADD foo.tar.gz /foo/
```

支援的格式有 `gzip, bzip2, xz`。

但這兩個功能是不能混用的，也就是說遠端下載的例子中的 `big.tar.xz` 是不會自動解壓縮的。

也就因為這些細部的功能如果一開始不知道的話很難直覺判斷執行解果，因此官方並不建議使用，如果想要從遠端下載檔案並解壓的話，可以用 `curl` 或 `wget` 搭配 `tar -x` 指令

```Dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL https://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

## 其他小細節

以下這些特性是兩者都有的

- `<dest>` 若為相對路徑，會以 `WORKDIR` 指令所指向的位置為起始點。
- `<src>` 必須要在 build context 裡面，因此 `COPY ../sth /sth` 是無效的，如果你沒有使用 Dockerfile 而是使用 STDIN 來輸入的話 (`docker build - < somefile`) 因為沒有 build context 所以 `COPY` 和 `ADD` 都是不能使用的。
- `<dest>` 如果結尾有 `/` 且位置不存在的話，會自動建立資料夾，但如果不是 `/` 結尾則會當作檔案寫入。
- `<src>` 可以是一個檔案或一個目錄，如果是目錄的話，會將目錄內的所有檔案都複製到 `<dest>` 裡面。
- `<src>` 可以指定多個檔案，但 `<dest>` 必須是一個目錄，如果 `<dest>` 不是一個目錄的話會造成非預期的後果。

### 小實驗

```Dockerfile
FROM ubuntu:18.04

COPY a b c d e /dir # 1
COPY ddd /dir1      # 2, ddd 是目錄
COPY ddd /dir2/     # 3, ddd 是目錄
COPY a /dir3        # 4
COPY a /dir4/       # 5
```

#### 結果

```
|-- dir (file)
|-- dir1
|   |-- a
|   |-- b
|   |-- c
|   |-- d
|   `-- e
|-- dir2
|   |-- a
|   |-- b
|   |-- c
|   |-- d
|   `-- e
|-- dir3 (file)
|-- dir4
    `-- a
```

可以注意到在複製資料夾的情況下，有無加結尾的 `/` 不影響結果 (#2, #3)，但是如果是複製檔案的話有加結尾的 `/` 會自動新增資料夾並複製檔案到該資料夾下，若沒有的話會當作新的檔名 (#4, #5)，而 #1 是展示多個來源檔案的情況下若目的沒有加結尾的 `/` 的情況，實測結果他會將 `e` 的內容複製進 `/dir`。

```
root@66b18a43ccd3:/# cat /dir
e
```
