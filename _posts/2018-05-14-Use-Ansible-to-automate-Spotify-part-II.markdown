---
layout: post
title:  "Use Ansible to automate Spotify - Part II"
date:   2018-05-14 12:00:00 +1000
categories: ansible
---

The last time I wrote a small introduction about how to use the Spotify authentication modules to create an **authentication token** and how this authentication token can be consumed by other Spotify Ansible modules e.g. `spotify_related_artists` (link)[{{ url.baseurl }}{% post_url 2018-04-27-Use-Ansible-to-automate-Spotify-part-I %}]. This time I am going to show you several use cases which combine several modules/roles.

* Create a playlist for a specific album
* Add related artists top tracks to a new playlist
* Play a specific playlist on a specific device

<!--excerpts-->

## Create a playlist for a specific album

Let's start with something easy. We want to search for an album and add all tracks to a new playlist. To realise this we need the following Ansible modules:

* spotify_auth
* spotify_auth_create_user_token
* spotify_search
* spotify_album
* spotify_user_playlists
* spotify_update_playlists

or the roles:

* authenticaton
* search
* album
* user_playlists
* update_playlists

**The Playbook for using the roles may look like this:**

Task 1: Create authentication token

Task 2: Search for album *Tenacious D* and response the first album you find

Task 3: Get all tracks from the album

Task 4: Create a new user playlist with the name *Tenacious D*

Task 5: Add all album tracks to the new playlist

![Create album playlist](https://i.imgur.com/8XZYASU.gif)

{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    temp_dest_file: "/tmp/tempfile.json"
    temp_playlist_file: "/tmp/temp_playlist.json"
    username: bloch-m
  tasks:
  - name: Create user authentication token
    include_role:
      name: authentication
    vars:
      config_file: "{{inventory_dir}}/group_vars/user.yaml"

  - name: search for album
    include_role:
      name: search
    vars:
      search_for: albums
      search_for_albums_name: Tenacious D
      search_result_output: short
      search_result_limit: 1
      search_dest_file: "{{ temp_dest_file }}"

  - name: Get album informations
    include_role:
      name: album
    vars:
      spotify_album_file: "{{ temp_dest_file }}"
      spotify_album_dest_file: "{{ temp_dest_file }}"
      spotify_album_output_format: short
      spotify_album_state: album_tracks

  - name: Create user playlist
    include_role:
      name: user_playlists
    vars:
      user_playlists_for_user: "{{ username }}"
      user_playlists_state: create
      user_playlists_name: Tenacious D
      user_playlists_dest_file: "{{ temp_playlist_file }}"

  - name: Add tracks to playlist
    include_role:
      name: update_playlists
    vars:
      update_playlist_state: add
      update_playlist_file: "{{ temp_playlist_file }}"
      update_playlist_track_file: "{{ temp_dest_file }}"
{% endraw %}
{% endhighlight %}

## Add related artists top tracks to a new playlist

Now let's do something even more complicated.

Let's assume we have one artists which music we enjoy. Passing this artists to the role `get_related_artists` will give us 20 related artists from the Spotify API. Since we really really enjoy the music from the first given artists the chances might be high, that we will enjoy the music from the related related artists so we may also want to know the related artists from the related artists. Now we will have as a maximum *20 * 20 artists*. Because we want to do something we want to get the 20 top tracks of each artists and add them to a own playlist. This will give us as maximum *20 * (20 * 20)* songs in a playlist with a lot of music we may like.

To realise this scenario we need the following Ansible modules:

* spotify_auth
* spotify_auth_create_user_token
* spotify_related_artists
* spotify_artists_top_tracks
* spotify_user_playlists
* spotify_update_playlists

or the roles:

* authenticaton
* get_related_artists
* get_artists_top_tracks
* user_playlists
* update_playlists

**The Playbook for using the roles may look like this:**

Task 1: Create authentication token

Task 2: Get related artists for artist *Young the Giant*

Task 3: Get related artists for all related artists found for *Young the Giant*

Task 4: Get top tracks for all related artists found

Task 5: Create a new user playlist with the name *Ansible created Playlist*

Task 6: Add all album tracks to the new playlist

![Create album playlist](https://i.imgur.com/AneW3GK.gif)

{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    artist: Young the Giant
    playlist_name: Ansible created Playlist
    temp_dest_file: "/tmp/tempfile.json"
    temp_playlist_file: "/tmp/temp_playlist.json"
    username: bloch-m

  tasks:
  - name: Create user authentication token
    include_role:
      name: authentication
    vars:
      config_file: "{{inventory_dir}}/group_vars/user.yaml"

  - name: "Get related artists for artist {{ artist }}"
    include_role:
      name: get_related_artists
    vars:
      get_related_artists_for: "{{ artist }}"
      get_related_dest_file: "{{ temp_dest_file }}"
      get_related_artists_output: short

  - name: "Get related-related artists for artist {{ artist }}"
    include_role:
      name: get_related_artists
    vars:
      get_related_artists_from_file: "{{ temp_dest_file }}"
      get_related_dest_file: "{{ temp_dest_file }}"
      get_related_artists_output: short

  - name: "Get top tracks from all found related artists."
    include_role:
      name: get_artists_top_tracks
    vars:
      get_top_tracks_from_artists_file: "{{ temp_dest_file }}"
      get_top_tracks_dest_file: "{{ temp_dest_file }}"
      get_top_tracks_output: short

  - name: Create user playlist
    include_role:
      name: user_playlists
    vars:
      user_playlists_for_user: "{{ username }}"
      user_playlists_state: create
      user_playlists_name: "{{ playlist_name }}"
      user_playlists_dest_file: "{{ temp_playlist_file }}"

  - name: Add tracks to playlist
    include_role:
      name: update_playlists
    vars:
      update_playlist_state: add
      update_playlist_file: "{{ temp_playlist_file }}"
      update_playlist_track_file: "{{ temp_dest_file }}"
{% endraw %}
{% endhighlight %}


## Play a specific playlist on a specific device

Since this is the last example why not celebrating it with a song ?

This example will play a specific song on a specific device. Since we don't know the **track URI** we need to search for the song. As we neither know the Device ID of the spotify device we query for all playable devices and filter them by name.
Once we have our song and device we will playback our song.

To realise this scenario we need the following Ansible modules:

* spotify_auth
* spotify_user_info
* spotify_player

or the roles:

* authenticaton
* user_info
* player

**The Playbook for using the roles may look like this:**

Task 1: Create authentication token

Task 2: Search for the track *Eye of the Tiger*

Task 3: Find all available spotify devices

Task 4: save device id of a specific spotify device

Task 5: Transfer playback to device

Task 6: set volume on device

Task 7: Pause any current plauback

Task 8: play track found earlier

![Create album playlist](https://i.imgur.com/NBTETkt.gif)
{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    username: bloch-m
    temp_dest_file: "/tmp/tempfile.json"
  tasks:
  - include_role:
      name: authentication
    vars:
      config_file: "{{inventory_dir}}/group_vars/user.yaml"

  - include_role:
      name: search
    vars:
      search_for: tracks
      search_for_tracks_name: Eye of the Tiger
      search_result_output: short
      search_result_limit: 1
      search_dest_file: "{{ temp_dest_file }}"

  - name: Get User information with default settings
    spotify_user_info:
      auth_token: "{{ auth_token }}"
      state: devices
      output_format: short
    register: sp_devices

  - set_fact:
      sp_device: "{{ item.device_id }}"
    with_items: "{{ sp_devices.result.devices }}"
    when: item.name == "MichaelBloch"

  - include_role:
      name: player
    vars:
      spotify_player_state: transfer_playback
      spotify_player_device_id: "{{ sp_device }}"
    register: sp_devices

  - include_role:
      name: player
    vars:
      spotify_player_state: volume
      volume_level_percent: 80
    register: sp_devices

  - include_role:
      name: player
    vars:
      spotify_player_state: pause
    register: sp_devices

  - name: "Ansible Spotify Play specific track"
    include_role:
      name: player
    vars:
      spotify_player_state: play
      spotify_track_file: "{{ temp_dest_file }}"
    register: sp_devices
{% endraw %}
{% endhighlight %}

In the next part I try to introduce a new module to use the Spotify *get recommendations based on seeds* and combine it with the module `spotify_track_data` to explore the wide world of Spotify to find new unexplored songs.

Have fun trying and testing.

Cheers!
