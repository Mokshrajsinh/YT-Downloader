import yt_dlp
import os
import subprocess

def list_formats(url):
    # Create a youtube-dl instance
    ydl_opts = {
        'noplaylist': True,  # Ensure we're downloading a single video
        'ignoreerrors': True,
        'quiet': True,  # Suppress verbose output during extraction
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        # Extract video information
        info = ydl.extract_info(url, download=False)
        
        # Get all available formats
        formats = info.get('formats', [])
        
        video_formats = []
        audio_formats = []
        format_index_map = {}  # Mapping printed index to actual format

        print("\nAvailable Video Formats (Video Only or Video+Audio):")
        for idx, fmt in enumerate(formats):
            if fmt.get('vcodec') != 'none':  # Video formats
                resolution = fmt.get('resolution', 'Unknown resolution')
                format_note = fmt.get('format_note', 'No info')  # Handle missing format_note
                print(f"{idx}: {fmt['format_id']} - {fmt['ext']} - {format_note} - {resolution}")
                video_formats.append((idx, fmt))
                format_index_map[idx] = len(video_formats) - 1  # Map printed index to actual list index

        print("\nAvailable Audio Formats (Audio Only):")
        for idx, fmt in enumerate(formats):
            if fmt.get('acodec') != 'none' and fmt.get('vcodec') == 'none':  # Audio-only formats
                audio_bitrate = fmt.get('abr', 'Unknown bitrate')
                print(f"{idx}: {fmt['format_id']} - {fmt['ext']} - {audio_bitrate}kbps")
                audio_formats.append((idx, fmt))
                format_index_map[idx] = len(audio_formats) - 1  # Map printed index to actual list index

        return video_formats, audio_formats, format_index_map

def download_format(url, format_id, output_name):
    ydl_opts = {
        'format': format_id,  # Download the specific format selected by user
        'outtmpl': output_name,
        'noplaylist': True,
        'ignoreerrors': True,
        'quiet': False,  # Set to True to suppress output
        'no_warnings': True,
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        try:
            ydl.download([url])
        except yt_dlp.utils.DownloadError as e:
            print(f"Download failed: {e}")

def merge_video_audio(video_file, audio_file, output_file):
    # Use ffmpeg to merge video and audio
    command = f'ffmpeg -i "{video_file}" -i "{audio_file}" -c:v copy -c:a aac "{output_file}"'
    try:
        subprocess.run(command, shell=True, check=True)
        print(f"Video and audio merged into {output_file}")
    except subprocess.CalledProcessError as e:
        print(f"Error during merging: {e}")

if __name__ == "__main__":
    url = input("Enter video URL: ")
    
    # List all available formats
    video_formats, audio_formats, format_index_map = list_formats(url)
    
    # Ask the user to choose a video format
    video_choice = int(input("\nEnter the number of the video format you want to download: "))
    
    if video_choice in format_index_map:
        actual_video_idx = format_index_map[video_choice]
        video_format_id = video_formats[actual_video_idx][1]['format_id']
    else:
        print("Invalid video choice!")
        exit()

    # Ask the user to choose an audio format
    audio_choice = int(input("\nEnter the number of the audio format you want to download: "))
    
    if audio_choice in format_index_map:
        actual_audio_idx = format_index_map[audio_choice]
        audio_format_id = audio_formats[actual_audio_idx][1]['format_id']
    else:
        print("Invalid audio choice!")
        exit()
    
    # Download the chosen video and audio
    video_output = 'video.mp4'
    audio_output = 'audio.m4a'
    
    print("\nDownloading video...")
    download_format(url, video_format_id, video_output)
    
    print("\nDownloading audio...")
    download_format(url, audio_format_id, audio_output)
    
    # Merge video and audio into a final output
    final_output = 'output_merged.mp4'
    print("\nMerging video and audio...")
    merge_video_audio(video_output, audio_output, final_output)

    # Cleanup: Remove the separate video and audio files
    if os.path.exists(video_output):
        os.remove(video_output)
    if os.path.exists(audio_output):
        os.remove(audio_output)
    
    print(f"\nDownload and merge completed! Merged file saved as: {final_output}")
