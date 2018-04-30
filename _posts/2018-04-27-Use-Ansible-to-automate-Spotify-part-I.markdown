---
layout: post
title:  "Use Ansible to automate Spotify - Part I"
date:   2018-04-29 00:00:00 +1000
categories: ansible
---

To find new music with Spotify I thought about to go a new way ... so why not using Ansible to automate Spotify ?

* The modules
* Step 1 - Prerequisites
* Step 2 - Authentication
* Step 3 - Let's do something

<!--excerpts-->

## The modules

To use Ansible to automate Spotify I developed 9 different Ansible modules which contacts the Spotifi API using the Python library Spotipy.

The modules including examples, roles and configuration can be found on Github [(Link)](https://github.com/PillPall/ansible-spotify-client).

Following Ansible modules are at the moment available:

| Module name                    | Description                                | Documentation |
| ------------------------------ | ------------------------------------------ | ------------- |
| spotify_auth                   | Module for the authentication process      |[Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_auth.md)|
| spotify_auth_create_user_token | Module for the user authentication process |[Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_auth_create_user_token.md)|
| spotify_player                 | Spotify Player to play a song and more.    |[Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_player.md)|
| spotify_search                 | Search in Spotify                          | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_search.md)|
| spotify_user_info              | Get user information                       | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_user_info.md)|
| spotify_user_playlists         | Search and create a user playlist          | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_user_playlists.md)|
| spotify_update_playlists       | Add or remove songs from a playlist        | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_update_playlists.md)|
| spotify_artists_top_tracks     | Get the top tracks of an artist            |[Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_artists_top_tracks.md)|
| spotify_related_artists        | Get related artists                        | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_related_artists.md)|
| spotify_track_data     | Get Audio features from a track or analyse tracks  | [Link](https://github.com/PillPall/ansible-spotify-client/blob/master/docs/Ansible_modules/spotify_artists_top_tracks.md)|

## Step 1 - Prerequisites

- **Download or clone the github project**

- **Spotify Client ID & Secret**

All modules are doing API Calls and to do API calls we need a **CLIENT ID and CLIENT SECRET**. You can go to [Spotify Developer(Link)](https://beta.developer.spotify.com/dashboard/applications) and create your own or use the one from my github example. If you create your own one don't forget to permit a redirect url. I recommend to use the default redirect url `http://mbloch.s3-website-ap-southeast-2.amazonaws.com`.

Once you have created your own keys you may want to update the inventory files `ansible/inventory/group_vars/public.yaml` & `ansible/inventory/group_vars/user.yaml`.

- **Spotipy**

All modules are using the python library Spotipy version 2.4.4 to connect to the Spotify API. Since the Github version contains several improvements and bug fixes which are not released on pip yet we need to install Spotipy from github:

{% highlight bash %}
> pip install git+https://github.com/plamere/spotipy.git --upgrade
{% endhighlight %}

## Step 2 - Authentication
After finishing the Prerequisites. Let's see how authentication is done as this is very important and quite complex.

Depending on the authentication process the modules require different parameters.

**Public authentication:**
* client_id
* client_secret

**User authentication:**
* client_id
* client_secret
* username
* redirect_uri
* scope

All Ansible modules require an **authentication_token**. To create an authentication token we need to use the modules `spotify_auth` & `spotify_auth_create_user_token`.

The `redirect_uri` is the URL where Spotify will redirect you once you allow the application access, more information can be found further down at **user authentication process**. The `scope` tells Spotify what your application is allows to access. More information and which scopes are available can be found here [(Link)](https://beta.developer.spotify.com/documentation/general/guides/scopes).

Depending on the use case we may want to use the public authentication or user authentication process.

#### Public authentication process

{% highlight yaml %}

connect to Spotify API
        |
        v
  pass parameters
        |
        v
  receive token
{% endhighlight %}

1. Connect to Spotify API and passing the parameters **client_id & client_secret**.
2. Receive **authentication token**

The public authentication process only requires the module `spotify_auth`. The **client_id & client_secret** can be set via configuration file i.e. `ansible/inventory/group_vars/public.yaml`:
{% highlight yaml linenos %}
client_id: a430d21d72594499a3aaee8dc9636a3f
client_secret: 7660b5c77ed34df48bac67beed2e100c
{% endhighlight %}

Playbook use:
{% highlight yaml linenos %}
{% raw %}
- name: Spotify public authentication via configuration file
  spotify_auth:
    config_file: "{{ inventory_dir }}/group_vars/public.yaml"
{% endraw %}
{% endhighlight %}

or we can set the parameters in a playbook:

{% highlight yaml linenos %}
- name: Spotify public authentication
  spotify_auth:
    client_id: a430d21d72594499a3aaee8dc9636a3f
    client_secret: 7660b5c77ed34df48bac67beed2e100c
{% endhighlight %}

#### User authentication process:

{% highlight yaml  %}
Search for cached credentials  ->  if exist use cached token
          |
          | if not
          |
          v
 connect to Spotify API
          |                        --------------
          v                       | spotify_auth |
    pass parameters                --------------
          |
          v
  allow application access &    
    receive API Code            
----------------------------------------------------------------
          |
          v                       --------------------------------
    pass API Code to             | spotify_auth_create_user_token |
      Spotify API                 --------------------------------
          |
          v
    receive token
{% endhighlight %}
1. Search for local cached credentials in `/tmp/.cache-USERNAME` and use if exist

2. if no cached credentials exists connect to Spotify API and pass parameters

3. **Allow application access** for your Spotiy Account & receive API Code

4. Pass **API Code** to Spotify API with module `spotify_auth_create_user_token`

5. Receive **authentication token**


As you can see the user authentication process is a bit more complex and requires the modules `spotify_auth` & `spotify_auth_create_user_token`. The required parameters can be set via configuration file i.e. **ansible/inventory/group_vars/user.yaml**:
{% highlight yaml linenos %}
client_id: a430d21d72594499a3aaee8dc9636a3f
client_secret: 7660b5c77ed34df48bac67beed2e100c
redirect_uri: http://mbloch.s3-website-ap-southeast-2.amazonaws.com
scope: playlist-modify-private,playlist-modify-public,user-read-currently-playing

{% endhighlight %}
Playbook use:
{% highlight yaml linenos %}
{% raw %}
- name: Spotify user authentication via configuration file
  spotify_auth:
    username: bloch-m
    config_file: "{{ inventory_dir }}/group_vars/user.yaml"
{% endraw %}
{% endhighlight %}

or we can set the parameters in a playbook:

{% highlight yaml linenos %}
- name: Spotify user authentication
  spotify_auth:
    username: bloch-m
    client_id: a430d21d72594499a3aaee8dc9636a3f
    client_secret: 7660b5c77ed34df48bac67beed2e100c
    redirect_uri: http://mbloch.s3-website-ap-southeast-2.amazonaws.com
    scope: playlist-modify-private,playlist-modify-public,user-read-currently-playing
{% endhighlight %}

The module `spotify_auth` checks if a cached authentication token exists and will return it. If there is no cached authentication token the module connects to the Spotify API and Ansible will open up your `default OS browser` and redirect you to the Spotify authentication page. Now you have to login to Spotify and allow the application Access to your Spotify. Once you have done this you will be redirected to the specified **redirect_uri**. If you are using the default redirect_uri copy the **API Code** as shown. If not you may need to copy the **API Code** from the address bar in your browser i.e. you need to copy the part after `?code=` e.g. `https://example.com/?code=ABCDEFG` copy `ABCDEFG`.

Once copied you need to pass it to the module `spotify_auth_create_user_token`.

{% highlight yaml linenos %}
{% raw %}
- name: Create user token from Spotify API user code
  spotify_auth_create_user_token:
    api_user_code: "ABCDEFG"
    username: bloch-m
    config_file: "{{inventory_dir}}/group_vars/user.yaml"

or

- name: Create user token from Spotify API user code
  spotify_auth_create_user_token:
    api_user_code: "ABCDEFG"
    username: bloch-m
    client_id: a430d21d72594499a3aaee8dc9636a3f
    client_secret: 7660b5c77ed34df48bac67beed2e100c
    redirect_uri: http://mbloch.s3-website-ap-southeast-2.amazonaws.com
    scope: playlist-modify-private,playlist-modify-public,user-read-currently-playing
{% endraw %}
{% endhighlight %}

#### Authentication role
To automate the authentication process as far as possible I created the role `authentication` which will set the authentication token as fact `auth_token`.

Role use in playbook:
{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    config_file: "{{ inventory_dir }}/group_vars/user.yaml"
    username: bloch-m
  roles:
    - authentication
{% endraw %}
{% endhighlight %}

The tricky part is passing the **API Code**, for creating the user authentication token, from the Ansible module `spotify_auth` to the module `spotify_auth_create_user_token`. To realise this, the role will tell Ansible to wait for Input while the Browser opens up the URL define in the `redirect_uri` option. Once the application is allowed copy the **API code** and go back to your terminal where Ansible is waiting for you and enter the **API Code**. After entering the API Code you will get an authentication token to use with the other Ansible modules like `spotify_update_playlists`.

Initial user authentication:
![User authentication](https://i.imgur.com/PZNaGGO.gif)

Cached User authentication:
![User authentication](https://i.imgur.com/xsAxAWY.gif)

Public authentication:
![Public authentication](https://i.imgur.com/qs4EVdi.gif)

## Step 3 - Let's do something

Now that we know how to authenticate we can finally do something. Let's start with something easy...

### Search with Ansible
To search in Spotify with Ansible we can use the role `search` which is using the module `spotify_search`. The module allows us to search for `artists, albums, playlists, tracks, artists & album, artists & track`.

Role use in playbook:
{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    config_file: "{{ inventory_dir }}/group_vars/public.yaml"
    search_for: artists
    search_for_artists_name: Young the Giant
    search_result_output: short
    search_result_limit: 20
  roles:
    - authentication
    - search
{% endraw %}
{% endhighlight %}

As you can see we first execute the role `authentication` and than executing the role `search`. The authentication role takes care of getting an authentication token from the Spotify API and due setting the authentication token as fact `auth_token` it can be used by all other roles included in the playbook.

Search artist Young the Giant:
![Search artist Young the Giant](https://i.imgur.com/xzOt9tf.gif)

### Access some User data
We can use the role `user_info` which use the module `spotify_user_info`. This module allows us to access user data like `current playback, recently played, users top artist, users top tracks and general user info`.

Role use in playbook:
{% highlight yaml linenos %}
{% raw %}
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    config_file: "{{inventory_dir}}/group_vars/user.yaml"
    username: bloch-m
    user_info_output: short
    user_info_limit: 50
    user_info_dest_file: "{{ playbook_dir }}/../files/{{ username }}_top_tracks.json"
    user_info_state: top_tracks
    user_info_time_range: medium_term
  roles:
    - authentication
    - user_info
{% endraw %}
{% endhighlight %}
{% raw %}
This playbook will save informations in short format about my most 50 played tracks within the time range `medium_term` to `{{ playbook_dir }}/../files/{{ username }}_top_tracks.json`. The saved file can than be consumed i.e. from the module `spotify_update_playlists` for adding those tracks to a own playlist.
{% endraw %}

Get users top tracks:
![Get users top tracks](https://i.imgur.com/LIooDPp.gif)

### Conclusions

Since the authentication process is a little bit complex I would recommend to always run the `authentication` role first.

Furthermore, I tried to build the modules as much as possible compatible to each other. Which means every module which has the capability to load information from a file can use the save results from other modules e.g. the saved output from the module `spotify_related_artists` can be consumed from the module `spotify_artists_top_tracks` to get all top tracks from related artists. This gives us the possibility to create more complex Ansible playbooks like add all top tracks from all found related artists to a new created playlist.

To get more information about the project, the Ansible modules and more examples visit the Github project [(Link)](https://github.com/PillPall/ansible-spotify-client).

More about constructing more complex playbooks is coming up in Part II.

Have fun trying and testing.

Cheers!
