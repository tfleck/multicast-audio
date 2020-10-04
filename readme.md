# Multicast Audio
This is a guide for how to build a modular multiroom audio system that supports Spotify & AirPlay using Raspberry Pis.

## Guide
---
### Server Pi
1. Make sure all software on your Raspberry Pi is up to date
    ```bash
    sudo apt update -y && sudo apt upgrade -y && sudo apt dist-upgrade -y && sudo apt autoremove -y && sudo apt clean -y && sudo apt autoclean -y
    ```
2. Get latest version of [badaix’s Snapcast *server*](https://github.com/badaix/snapcast/releases/latest). You can use wget to download it from the command line like this (for v0.17.1 at the time of writing). Replace v0.17.1 with the latest version number in the URL below
    ```bash
    wget https://github.com/badaix/snapcast/releases/download/v0.19.0/snapserver_0.19.0-1_armhf.deb
    ```

3. Unpack and install the Snapcast server
    ```bash
    sudo dpkg -i snapserver_0.19.0-1_armhf.deb
    ```

4. Install any missing dependencies
    ```bash
    sudo apt-get -f install
    ```

5. Download & install the latest version of [dtcooper's Raspotify](https://github.com/dtcooper/raspotify#easy-installation)
    ```bash
    curl -sL https://dtcooper.github.io/raspotify/install.sh | sh
    ```

6. Edit the Raspotify default configuration
    ```bash
    sudo nano /etc/default/raspotify
    ```
    Change the device name (this is what will be displayed in Spotify Connect)
    ```
    #DEVICE_NAME=”raspotify” -> DEVICE_NAME=”<your name here>”
    ```
    Change bitrate to 320 kbps (Spotify’s “high quality” premium streaming)
    ```
    #BITRATE=”160” -> BITRATE=”320”
    ```
    Configure output to the Snapserver FIFO pipe (this will allow all Snapcast clients to play the audio)
    ```
    #BACKEND_ARGS="--backend alsa" -> BACKEND_ARGS="--backend pipe --device /tmp/snapfifo"
    ```

7. Restart Raspotify daemon to apply changes
    ```bash
    sudo systemctl restart raspotify
    ```

8. Edit the Snapserver default configuration
    ```bash
    sudo nano /etc/snapserver.conf
    ```
    Change the sampling rate from 48kHz to 44.1kHz (otherwise the playback doesn't sound right)
    ```
    sampleformat = 48000:16:2 -> sampleformat = 44100:16:2
    ```

9. Restart Snapserver to apply changes
    ```bash
    sudo systemctl restart snapserver
    ```
    
10. Download & install the latest version of [mikebrady's Shairport-Sync](https://github.com/mikebrady/shairport-sync) for AirPlay support (optional)
    ```bash
    sudo apt-get install build-essential git xmltoman autoconf automake libtool \
    libpopt-dev libconfig-dev libasound2-dev avahi-daemon libavahi-client-dev libssl-dev libsoxr-dev
    git clone https://github.com/mikebrady/shairport-sync.git
    cd shairport-sync
    autoreconf -fi
    ./configure --sysconfdir=/etc --with-alsa --with-soxr --with-avahi --with-ssl=openssl --with-systemd --with-pipe
    make
    sudo make install
    ```
    
11. Edit default Shairport-Sync configuration
    ```bash
    sudo nano /etc/shairport-sync.conf
    ```
    Change the device name
    ```
    //    name =”%H”; -> name = “<your name here>”;
    ```
    Change the output backend to snapserver fifo pipe
    ```
    //     output_backend = "alsa"; -> output_backend = “pipe”;
    ```
    Change output pipe location
    ```
    //    name = "/path/to/pipe"; -> name = “/tmp/snapfifo”;
    ```

12. Start Shairport-Sync
    ```bash
    sudo systemctl start shairport-sync
    ```
    
13. Enable services to start on boot
    ```bash
    sudo systemctl enable snapserver
    sudo systemctl enable raspotify
    sudo systemctl enable shairport-sync
    ```    

### Client Pi(s)
1. Get the latest version of [badaix's Snapcast *client*](https://github.com/badaix/snapcast/releases/latest). Same as with the server package, replace the version number in the URL below with the latest version number on GitHub.
    ```bash
    wget https://github.com/badaix/snapcast/releases/download/v0.19.0/snapclient_0.19.0-1_armhf.deb
    ```

2. Unpack and Install Snapclient
    ```bash
    sudo dpkg -i snapclient_0.19.0-1_armhf.deb

    ```

3. Install any missing dependencies
    ```bash
    sudo apt-get -f install
    ```

### Last Steps
- Connect your speakers to the 3.5mm jack on the Raspberry Pi.
- Open the Spotify app, and go to the “Devices Available” menu. You should see a new device with the name you set earlier. If you select and start playing something, you should hear it come out of all the connected speakers!
- Further configuration can be done with Raspotify and Snapcast to tailor the system to your exact needs. Check out their respective GitHub repositories for more information.
- If you’re using a hifiberry dac/amp, follow this [guide to setup that as the default output instead](https://www.hifiberry.com/docs/software/configuring-linux-3-18-x/)

## Troubleshooting
- Make sure raspotify and shairport-sync have permissions to the /tmp/snapfifo directory
- If the volume is too loud, try editing the raspotify config to remove the --linear-volume flag, and reduce the --initial-volume=100 to something lower
- In the raspotify config, you can also remove the --enable-volume-normalisation flag if you want, that’s up to your personal preference

## Technical Information
Raspotify is used to impersonate a Spotify Connect speaker, so that your Spotify app will detect it on your network and allow you to control it just like any other Spotify device.

Raspotify writes the raw audio to a buffer rather than an actual audio device to accomodate Snapcast without having to directly integrate both pieces of software.

Snapcast server takes the audio that is written into the buffer, and sends it out over the network. Snapcast uses continuous time synchronization so the audio is played back by all clients at the exact same time. As many clients as you want can connect to the server and receive the audio chunks.

The advantage of this approach is that you can use things other than Raspotify to feed Snapcast, such as [shairport-sync](https://github.com/mikebrady/shairport-sync). Anything that is written into the buffer gets played by all clients, regardless of which application wrote it there. The downside of this system is that you have to control which applications write to the buffer at any one time, if multiple applications write at the same time, you get some really strange sounding audio.

On the client side, you can control the audio volume individually. Or control the playback after the Pi's entirely. You could have a client Pi in each room of you house attached to speakers through an amplifier. That way the Pi can run 24/7, always outputting whatever is being played, and you control on/off and volume purely by the settings for the amplifier.

The architecture of this system is intended to clearly separate each link in the chain to make it easy to modify/upgrade to each individual deployment.

## Acknowledgements/Resources
This project would not be possible without people out there making awesome open source software
- https://github.com/badaix/snapcast
- https://github.com/dtcooper/raspotify
- https://github.com/librespot-org/librespot
- https://digitmind.net/en/build-your-own-multiroom-audio-system-with-raspberry-pi/










