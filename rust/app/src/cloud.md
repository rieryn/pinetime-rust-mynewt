# Build PineTime Firmware in the Cloud with GitHub Actions

![PineTime Firmware Programming vs Fortnite](https://lupyuen.github.io/images/cloud-title.jpg)

Programming the firmware of our gadgets (like [__PineTime Smart Watch__](https://wiki.pine64.org/index.php/PineTime)) has always been cumbersome...

1. Get a proper computer _(Windows tends to be problematic)_

1. Install the right tools and libraries to cross-compile our firmware _(Depends on our operating system)_

1. If the build fails, tweak the build scripts _(It's probably just Windows)_

1. If the build still fails... __We're stuck!__ _(Good Luck!)_

Nowhere as fun as a game like Fortnite!

_Can Firmware Programming be as fun as Fortnite?_

Well Fortnite runs in the Cloud on massive servers...

_Can we build our firmware in the Cloud?_

_In under 2 minutes?_

_Without any computer setup!_

Yes we can!

Today we'll learn [__GitHub Actions__](https://github.com/features/actions) for building [__FreeRTOS Firmware__](https://github.com/JF002/Pinetime) for [__PineTime Smart Watch__](https://wiki.pine64.org/index.php/PineTime) in the GitHub Cloud.

It's __Fully Automated__...

1. __Check in our updated source files__ to GitHub

1. Wait __2 Minutes__

1. Out comes a piping-hot __New Firmware Image__ for testing on PineTime!

_(Feels like a Microwave!)_

We'll make PineTime Programming as enjoyable as Fortnite... But less violent... And in 3D!

# Create A GitHub Fork

_(Nope no spoon!)_

1.  __Create a free GitHub Account__ if we haven't got one...

    ▶️ &nbsp; [`github.com`](https://github.com/)

1.  __Browse to the GitHub Repository__ for the PineTime Firmware...

    ▶️ &nbsp; [`github.com/JF002/Pinetime`](https://github.com/JF002/Pinetime)

    Here's the complete Source Code for the FreeRTOS PineTime Firmware.

1.  Click the `Fork` button at top right...

    ![Create a fork](https://lupyuen.github.io/images/cloud-fork.png)

1.  This creates a __Fork__ of the PineTime Repository under our GitHub Account...

    ![Created the fork](https://lupyuen.github.io/images/cloud-fork2.png)

    The URL looks like this...
    
    ```
    https://github.com/ACCOUNT_NAME/Pinetime
    ```

1.  The Fork contains __our own copy__ of the entire Source Code for the PineTime Firmware... Ready for us to make any updates!

    GitHub helpfully __tracks updates to our Fork,__ so that one day we may submit a __Pull Request__ to sync our updates (only the useful ones) back to the original PineTime Repository.

    And we may also __Pull Updates__ from the original PineTime Repository and apply them to our Fork.

    That's how Open Source projects are maintained!

Read on to learn how we add GitHub Actions to our Fork to build the firmware automagically...

# Add GitHub Actions

1.  In our Fork on GitHub, click `Actions`

1.  Click `Skip this and set up a workflow yourself`

1.  GitHub brings us to a page to edit `.github/workflows/main.yml`

1.  Open a new web browser tab. 

    Browse to this page...

    [`github.com/pinetime-lab/.github/workflows/main.yml`](https://raw.githubusercontent.com/lupyuen/pinetime-lab/master/.github/workflows/main.yml)
    
1.  Copy the contents of this page. 

    Paste the contents into the earlier page: `.github/workflows/main.yml`

# Build Our Firmware

TODO

# Test Our Firmware

TODO

If you could... With your kind permission... Please post to Twitter and/or Mastodon a pic of your PineTime with the new firmware.

Tag the post with `#PineTime` so we know that building PineTime Firmware in the Cloud works OK for you. Thanks! :-)

# How It Works

TODO

https://github.com/lupyuen/pinetime-lab/blob/master/.github/workflows/main.yml

```yaml
# GitHub Actions Workflow to build FreeRTOS Firmware for PineTime Smart Watch
# Based on https://github.com/JF002/Pinetime/blob/master/doc/buildAndProgram.md

# Name of this Workflow
name: Build PineTime Firmware

# When to run this Workflow...
on:

  # Run this Workflow when files are updated (Pushed) in the "master" Branch
  push:
    branches: [ master ]
    
  # Also run this Workflow when a Pull Request is created or updated in the "master" Branch
  pull_request:
    branches: [ master ]

# Steps to run for the Workflow
jobs:
  build:

    # Run these steps on Ubuntu
    runs-on: ubuntu-latest

    steps:
    - name: Install cmake
      uses: lukka/get-cmake@v3.18.0

    - name: Check cache for Embedded Arm Toolchain arm-none-eabi-gcc
      id:   cache-toolchain
      uses: actions/cache@v2
      env:
        cache-name: cache-toolchain
      with:
        path: ${{ runner.temp }}/arm-none-eabi
        key:  ${{ runner.os }}-build-${{ env.cache-name }}
        restore-keys: ${{ runner.os }}-build-${{ env.cache-name }}

    - name: Install Embedded Arm Toolchain arm-none-eabi-gcc
      if:   steps.cache-toolchain.outputs.cache-hit != 'true'  # Install toolchain if not found in cache
      uses: fiam/arm-none-eabi-gcc@v1.0.2
      with:
        # GNU Embedded Toolchain for Arm release name, in the V-YYYY-qZ format (e.g. "9-2019-q4")
        release: 8-2019-q3
        # Directory to unpack GCC to. Defaults to a temporary directory.
        directory: ${{ runner.temp }}/arm-none-eabi

    - name: Check cache for nRF5 SDK
      id:   cache-nrf5sdk
      uses: actions/cache@v2
      env:
        cache-name: cache-nrf5sdk
      with:
        path: ${{ runner.temp }}/nrf5_sdk
        key:  ${{ runner.os }}-build-${{ env.cache-name }}
        restore-keys: ${{ runner.os }}-build-${{ env.cache-name }}
          
    - name: Install nRF5 SDK
      if:   steps.cache-nrf5sdk.outputs.cache-hit != 'true'  # Install SDK if not found in cache
      run:  cd ${{ runner.temp }} && curl https://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v15.x.x/nRF5_SDK_15.3.0_59ac345.zip -o nrf5_sdk.zip && unzip nrf5_sdk.zip && mv nRF5_SDK_15.3.0_59ac345 nrf5_sdk

    - name: Checkout source files
      uses: actions/checkout@v2

    - name: Show files
      run:  set ; pwd ; ls -l

    - name: CMake
      run:  mkdir -p build && cd build && cmake -DARM_NONE_EABI_TOOLCHAIN_PATH=${{ runner.temp }}/arm-none-eabi -DNRF5_SDK_PATH=${{ runner.temp }}/nrf5_sdk -DUSE_OPENOCD=1 ../
      
    - name: Make
      # For debugging builds, remove option "-j" for clearer output. Add "--trace" to see details.
      run:  cd build && make -j pinetime-app
      
    - name: Find output
      run:  find . -name pinetime-app.out

    - name: Upload built firmware
      uses: actions/upload-artifact@v2
      with:
        # Artifact name (optional)
        name: pinetime-app.out
        # A file, directory or wildcard pattern that describes what to upload
        path: build/src/pinetime-app.out
      
# Embedded Arm Toolchain and nRF5 SDK will only be cached if the build succeeds.
# So make sure that the first build always succeeds, e.g. comment out the "Make" step.
```

Future
Bluetooth flashing

FYI our plans for putting in Continuous Integration with GitHub Actions

so whenever we check in changes to the source code, it will auto build the firmware

there will probably be 2 versions of this Continuous Integration...


(1) For maintaining the central PineTime firmware


(2) For newbies to build their PineTime firmware automatically in the cloud, without any software to be installed on their PCs


This is super cool, that makes so much so simple, no hassle with finding the right version of every software!

but of course there will be folks who dont like to build things in a cloud 

but the good news that you can see exactly what steps we use to build firmware in the cloud

and replicate on your own pc

and with actual logs to compare the results

so its super educational yay!


# What's Next?

[Check out my RSS Feed](https://lupyuen.github.io/rss.xml)