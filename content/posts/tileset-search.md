+++
title = 'Pixel Art Search'
date = 2023-12-29T12:29:11+02:00
draft = true
tags = ["GameDev", "Image Processing"]
+++

Those familiar with 2D game development probably already know what a tileset is. Instead of creating every scene from scratch, you reuse small square images (tiles) and arrange them in a grid to build a game world. It's like digital LEGO: you have a collection of pieces that you can combine in endless ways. Here's an example:
<!--more-->

![Modern Exteriors screenshot](/images/tileset-search/modern-exteriors.png)

Game engines like Godot or Unity are optimized for this approach. Rather than loading thousands of individual image files, they prefer working with "tilesets" - large images containing many tiles arranged in a grid. It's more efficient and easier on memory.

A while back I bought a [tileset pack on itch.io](https://limezu.itch.io/modernexteriors) with more than 5,000 individual tiles. It came in two formats: individual files (tagged, great for browsing) and combined tilesets (what you actually use in your game).

After using it for a while, I realised I had a problem: having tagged individual files is nice, but to use them in the editor you need to know where a tile is in the tileset. While there is some logic to how they're organzied, the reality is that with thousands of tiles and no coordinate system, often the only way to find and individual tile is to play a long session of [Where's Wally](https://en.wikipedia.org/wiki/Where%27s_Wally%3F).

After checking [xkcd](https://xkcd.com/1205/) to justify my procrastination, I decided to write a tool to solve this problem. While I've probably only recouped about half of the time I spent coding it, over a dozen other developers have found it useful, so I'm calling it a win.

Anyway, it was a fun weekend project and I've decided to document it, in case I need to update it in the future or adapt it to a different tileset, as I'm pretty sure I'll have forgotten how it works by then.

## The Challenge

The individual tiles come with descriptive filenames like "modern_house_window_blue.png". The folders are named by theme. The missing link for an efficient workflow is matching the individual files to their locations in the tilesets.

## How It Works

The solution is basically writing some code to handle the "Where's Wally" step by matching tiles with tilsets using Python and OpenCV.

The processing happens in three stages:

### 1. Asset Loading and Tagging
- Recursively scan directories for both tilesets and the different types of tiles (regular, animated, character sprites).
- Parse the file name and directory to get a list of tags.
- Filter out irrelevant data like file extension.

### 2. Matching
The next step is to scan through tilesets in a sliding window, checking if each of the 5256 individual tiles in the asset pack can be found in any of the 26 tilesets.
```python
th, tw, _ = tileset.shape
sh, sw, _ = single.shape
for j in range(size - sh, th, size):
    for i in range(size - sw, tw, size):
        ty0, ty1 = max(0, j), min(th, j + sh)
        tx0, tx1 = max(0, i), min(tw, i + sw)
        t = tileset[ty0:ty1, tx0:tx1]
        sy0, sy1 = max(0, -j), min(sh, th - j)
        sx0, sx1 = max(0, -i), min(sw, tw - i)
        s = single[sy0:sy1, sx0:sx1]
        am = alpha_mask[sy0:sy1, sx0:sx1]
        score = self._jaccard_index(s, t, am)
        if score > threshold:
            t_region = tx0, ty0, tx1 - tx0, ty1 - ty0
            s_region = sx0, sy0, sx1 - sx0, sy1 - sy0
            yield t_region, s_region, (score - threshold) / (1 - threshold)
```
Tiles are 16x16, but some individual images are actually multiple tiles, that's why the window size can't be fixed and needs to be adjusted.

A quick run with an empty comparison shows that this brute search approach will take 431,177,494 comparisons. Tiles are _at least_ 16x16, each has 4 bytes because images have an alpha channel, which gives us a lower bound of 32GB of data to process. If we don't make the comparisons blazing fast processing everything could take a while. The good news is that we're just sampling 150 MB of images so everthing fits easily in memory.

OpenCV for Python relies on Numpy so we can use that to calculate the Jaccard similarity index (ratio of non-transparent pixels that match in both images):
```python
@staticmethod
def _jaccard_index(a: cv2, b: cv2, am: cv2) -> float:
    """Return an index of how much a matches b,
    where 0.0 is no match, and 1.0 is an exact match."""
    if denominator := np.count_nonzero(am):
        comp = (a == b).min(axis=2)
        return np.count_nonzero(comp & am) / denominator
    return 0.0
```
We can also spread the load accross all available CPUs in a couple lines using Python's `multiprocessing`:
```python
with Pool() as pool:
    results = pool.imap_unordered(self._search, args, 32)
    results = list(tqdm(results, total=len(args)))
```
Another low-hanging optimization is to skip transparent areas:
```python
if np.count_nonzero(alpha_mask) == 0:
    return
```
This brings the script under half an hour on my desktop. I'm sure it can be done orders of magnitude faster, but it's good enough for what I wanted.


### 3. HTML
- When the script finds a match, it saves the tile and it's location in the matching tileset.
- The index is saved as json, an HTML is generated to filter and display the tiles in the browser:

![Tileset Search Screenshot](/images/tileset-search/screenshot2.png)

The tool outputs a JSON file mapping each individual tile to its exact location in the tilesets. Now instead of scanning through massive sprite sheets by eye, developers can search by keywords like "window" or "brick" and instantly see where to find matching tiles.
