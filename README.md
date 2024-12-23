# Convert video to gif using ffmpeg

## Introduction

This is a demonstration of how to convert a video to a gif file using ffmpeg. This usage focuses on reducing the size of the gif file while preserving the quality of the video. You can only read the example section and be good to go. If you want to learn how to adjust the parameters, you can read the rest of the document. Good luck!

## Dependencies

```bash
sudo apt install ffmpeg # for ubuntu or debian
brew install ffmpeg # for mac
```

## Local setup for reducing mp4 file size

After installing ffmpeg, you can add the following functions to your `.bashrc` or `.bash_profile` file (or `.zsh` and etc.).

The input here is the times you want to reduce the resolution of the video. For example, if you want to reduce the resolution by 2 times. You also provide the input video file and the output file. Examples are shown below.

```bash
reduce_by() {
    if [ "$#" -ne 3 ]; then
        echo "Usage: reduce_by_<NUMBER> <input> <output>"
        return 1
    fi

    NUMBER=$1
    INPUT=$2
    OUTPUT=$3

    # Get input video width and height
    # Make sure the new width and height are divisible by 2
    SCALE="scale=trunc(iw/${NUMBER}/2)*2:trunc(ih/${NUMBER}/2)*2"

    echo "Reducing resolution by $NUMBER times..."
    ffmpeg -i "$INPUT" -vf "$SCALE" -c:v libx264 -crf 23 -preset fast -c:a aac -b:a 128k "$OUTPUT"
    echo "Done! Output saved to $OUTPUT"
}

# Create alias for specific reduce_by functions
# Feel free to add more if you need. Not sure if reducing by any more is useful though.
for num in {1..5}; do
    alias reduce_by_$num="reduce_by $num"
done
```

I would also recommend adding the following functions to your `.bashrc` or `.bash_profile` file (or `.zsh` and etc.).

This will allow you to run `video_size input.mp4` to get the width and height of the video. It is useful when you want to check the output video of the `reduce_by` to confirm it is functional.

```bash
alias video_size="ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0"
```

## Actually converting the video to gif

After setting up the function to reduce your 4k 60fps video to a smaller resolution, you can now convert the video to a gif file.

This is a two step command. The first step is to generate a palette file from the video. The second step is to use the palette file to generate the gif file.

This approach will ensure that the gif file is of high quality and small size by picking out the specific colors in the video. It is a great optimization technique provided for free by ffmpeg.

Step 1: Generate palette file

```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,palettegen" palette.png
```

Step 2: Generate gif file

```bash
ffmpeg -i input.mp4 -i palette.png -lavfi "fps=10,scale=320:-1:flags=lanczos,paletteuse=dither=bayer:bayer_scale=5" output.gif
```

The variables to play around here are the `fps`, `scale`, and `bayer_scale`. The `fps` is the frame rate of the gif file. I find 10 to be good enough. The `scale` is the resolution of the gif file. The `bayer_scale` is the dithering scale. The higher the number, the more colors are used in the gif file.

## Example

Step 1: Check the file size of the video file.

```bash
ls -lrth input.mp4
```

Step 2: If your size in megabytes increases what you would prefer, use the reduce_by function to reduce the resolution of the video by 2 times. Optionally use the video_size function to check the width and height of the video. For autoplaying gifs on a website homepage, I sugges to aim for an mp4 that is less than 1 megabytes.

```bash
video_size input.mp4
reduce_by_2 input.mp4 output.mp4
video_size output.mp4
```

The output of the video_size function should be the width and height of the video. If the width and height are reduced by 2 times, then the function is working as expected.

Step 3: Obtain the palette file.

```bash
ffmpeg -i output.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,palettegen" palette.png
```

Step 4: Generate the gif file.

```bash
ffmpeg -i output.mp4 -i palette.png -lavfi "fps=10,scale=320:-1:flags=lanczos,paletteuse=dither=bayer:bayer_scale=5" output.gif
```

Step 5: Check the file size of the gif file.

```bash
ls -lrth output.gif
```

If the gif file is too large, you can reduce the fps or scale of the gif file. You can also increase the bayer_scale to reduce the number of colors in the gif file. Another approach is to reduce the resolution of the video file further. So, go back to step 2 and reduce the resolution by 2 times again.

## Conclusion

For autoplaying gif's on a website, I would sugges to aim for less than 1 megabytes. This is a good size for a gif file, and ensures your website has a good loading time, making it feel snappy and responsive. If you have any questions, feel free to ask. Good luck!

FFMPEG is a powerful tool that can do a lot of things. This is just one of the many things it can do. If you are interested in learning more about ffmpeg, I would recommend reading the documentation. It is very detailed and provides a lot of examples.

## References

- [ffmpeg documentation](https://ffmpeg.org/ffmpeg.html)
