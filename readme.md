# Multicast Audio
Overview information will go here.

## Guide
---
### Server Pi
1. Make sure all software on your Raspberry Pi is up to date
    ```bash
    sudo apt update -y && sudo apt upgrade -y && sudo apt dist-upgrade -y && sudo apt autoremove -y && sudo apt clean -y && sudo apt autoclean -y
    ```
2. Get latest version of [badaix’s Snapcast *server*](https://github.com/badaix/snapcast/releases/latest). You can use wget to download it from the command line like this (for v0.17.1 at the time of writing). Replace v0.17.1 with the latest version number in the URL below
    ```bash
    wget https://github.com/badaix/snapcast/releases/download/v0.17.1/snapserver_0.17.1-1_armhf.deb
    ```

3. Unpack and install the Snapcast server
    ```bash
    sudo dpkg -i snapserver_0.17.1-1_armhf.deb
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

### Client Pi(s)
1. Get the latest version of [badaix's Snapcast *client*](https://github.com/badaix/snapcast/releases/latest). Same as with the server package, replace the version number in the URL below with the latest version number on GitHub.
    ```bash
    wget https://github.com/badaix/snapcast/releases/download/v0.17.1/snapclient_0.17.1-1_armhf.deb
    ```

2. Unpack and Install Snapclient
    ```bash
    sudo dpkg -i snapclient_0.17.1-1_armhf.deb
    ```

3. Install any missing dependencies
    ```bash
    sudo apt-get -f install
    ```

### Last Steps
- Connect your speakers to the 3.5mm jack on the Raspberry Pi.
- Open the Spotify app, and go to the “Devices Available” menu. You should see a new device with the name you set earlier. If you select and start playing something, you should hear it come out of all the connected speakers!
- Further configuration can be done with Raspotify and Snapcast to tailor the system to your exact needs. Check out their respective GitHub repositories for more information.

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










