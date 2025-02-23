criteria for LLMS:

```
i need help finding the correct web-access file streamer/server. this is my criteria

- the service will in docker on a separate host with a NAS mounted drive
- all of the data is on a NAS at my house (users will seldom use their own 'storage' but instead access the NAS)
- I want users to be able to self-register (but maybe I can whitelist emails, or APPROVE registered users?)
- for my family only, about 20 people
- i want it to be simple (nextcloud seemed a bit much and clunky)
- link sharing would be nice but not a must-have
- access through web necessary (I like filebrowse)
- i would like desktop clients so we can mount it as a drive
- I would like to selectively picked directories in the mounted drive to be availible offline. we store our CAD projects there and if there is a local copy the programs work much better and then they sync to the server (but we dont want to force the ENTIRE NAS storage to users desktops, only CAD directories)
- file locking would probably need to be necessary so CAD files are not opened at the same time (some CAD programs do this themselves)
- we *do not* need collaboration features
- we will not be syncing photos from devices like cellphones
- i would like it to be simple and easy to manage

Options I have seen are Nextcloud, Owncloud (OCIS), Filebrowse, Seafile. what other options do I have? which would you recommend? I do not like clunki-ness of nextcloud, i dont like the feature locks of seafile, i like filebrowse but it seems to not do selective directory syncing on desktops? I like the modern GO platform of OCIS but think some features are missing? I am not opposed to 3rd party webDAV clients for mobile and desktop though would like a native app for my family members. I understand I will have to make some compromises so recommend what you think fits my use case. Give specifics too, like if you think nextcloud, should i run an AIO container or separate nextcloud container and database container? This will run on a big family server in a big docker-compose with 30+ containers. There will be backup plans in place to create backups locally, offsite, and CRITICAL data will be backup with a provider like backblaze or something
```
