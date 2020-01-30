GDPR archives also contain all medias uploaded to Twitter, linked to tweets and DMs. 

**Media access is restricted to GDPR archives !**

Please know first that medias linked to tweets can be obtained through their URL (always public, even if the account is protected). 
Direct message medias are protected with OAuth, so you should use a Twitter application in order to get medias via API.

Otherwise, `twitter-archive-reader` can find for you the right file linked to a direct message, tweets or more with methods available on the `MediaArchive` instance, located on the `.medias` property of `TwitterArchive`.
Some methods available on this object are made to facilitate access to tweet and DM medias.

```ts
import TwitterArchive, { MediaArchive } from 'twitter-archive-reader';
 
const archive = new TwitterArchive('filename');
await archive.ready();

const medias = archive.medias;
```

## Warning 
In archives made between **June 2019** and **December 2019**, media files were zipped inside the archive.
In this case, `twitter-archive-reader` must extract the ZIP from the original archive to read its content.

This cause a huge overhead when first accessing selected media archive, and may be fatal for RAM-limited systems with very big archives.

To know if medias are zipped inside the archive, you can use the `.is_medias_zipped` property of `MediaArchive`.

```ts
if (medias.is_medias_zipped) {
  // Try to avoid media getters through archive
}
```

## Mapping between media archives and `MediaArchiveType` enumeration

An enumeration (`MediaArchiveType`) is available to reference each supported folder by `MediaArchive`.
Enumeration items are used with `.get()` and `.list()` methods.

```ts
enum MediaArchiveType {
  SingleDM, GroupDM, Moment, Tweet, Profile
}
```

Each enum is respectively linked to `direct_message_media`, `direct_message_group_media`, `moments_media`, 
`tweet_media` and `profile_media` folders in GDPR archives.

You can import it as a component `twitter-archive-reader` package.
```ts
import { MediaArchiveType } from 'twitter-archive-reader';
```

## Get and list files through media archive

A bunch of methods on `MediaArchive` allow raw and easy access to medias from various sources.

Listing and getting files is totally asynchronous, so every method returns a `Promise`.

When you try to get a media, if it is not found **(that can happen !)**, the `Promise` will be *rejected*.

**Note**:
On each method that extract file(s), they can be returned in two formats: `Blob` or `ArrayBuffer`.
Choice is controlled by the last parameter on each file getter method, `as_array_buffer`.

By default, with `as_array_buffer === undefined`, the return type will be `Blob` 
if the platform supports it (generally, in browsers), otherwise it will be `ArrayBuffer` (in Node.js).

You can force return type by setting `as_array_buffer` to `true` (=> `ArrayBuffer`) or `false` (=> `Blob`).
For type safety, it is recommanded to always explicitely set this parameter.

### List available files

You can get available files on a directory with `.list(archive_type)` method.

```ts
// Get a list of files in tweet_media directory (medias related to tweets)
const filenames = await archive.medias.list(MediaArchiveType.Tweet);
```


### Get a file by its name

By using `.get(archive_type, filename, as_array_buffer)`, you can get a filename in the media archive of your choice.

```ts
const file = await archive.medias.get(MediaArchiveType.Tweet, filenames[0], /* as_array_buffer */);
```


### Get medias from a direct message

When you have a direct message, you don't -directly- have the related media filenames attached to it.

Some helpers are here to guide you: `.ofDm()` and `.fromDmMediaUrl()`.

```ts
const dm = archive.messages.single('dm_id');

// You can get directly all the medias of a DM
const medias_of_my_dm: (Blob | ArrayBuffer)[] = await archive.medias.ofDm(dm, /* as_array_buffer */);

// ...or you can get one of them via a media URL
if (dm.mediaUrls.length) {
  const media_1 = await archive.medias.fromDmMediaUrl(
    dm.mediaUrls[0], 
    /* is_group */, // You should specify here if a message come or not from a group conversation
    /* as_array_buffer */
  );
}
```

### Get medias from tweets
The same kind of methods exists for tweets.

Some helpers are here to guide you: `.ofDm()` and `.fromTweetMediaEntity()`.

```ts
const tweet = archive.tweets.all[0];

// You can get directly all the medias of the tweet
const medias_of_tweet: (Blob | ArrayBuffer)[] = await archive.medias.ofTweet(tweet, /* as_array_buffer */);

// ...or you can get one of them via a media entity
if (tweet.extended_entities || tweet.entities) {
  // Always try to use extended entities instead of classic entities
  const m_entities = (tweet.extended_entities || tweet.entities).media;

  if (m_entities && m_entities.length) {
    const media_file = archive.medias.fromTweetMediaEntity(m_entities[0]);
  }
}
```

### Get user profile picture and banner

Helpers `.getProfilePictureOf()` and `.getProfileBannerOf()` allows you to get easily profile medias.

The both methods take a `UserData` instance in parameter. Usally, this is `archive.user`.

```ts
const [profile, header] = await Promise.all([
  archive.medias.getProfilePictureOf(archive.user),
  archive.medias.getProfileBannerOf(archive.user)
]);
```

### Use case example of media getters

On browser, by default, getters returns `Blob` on browsers. 
This facilitate usage inside `<img>` and `<video>` tags.

```ts
const msg = archive.messages.single('dm_id');

/* Browser */
// Get the image
const blob = await archive.medias.fromDmMediaUrl(msg.mediaUrls[0], false, false) as Blob;

// Create a URL and set it as img
const url = URL.createObjectURL(blob);
document.querySelector('img').src = url;
```

On Node.js, getters returns `ArrayBuffer` by default.
You can also force it using `true` as last parameter on file getter methods.

```ts
/* Node.js */
// Get the image
const array_buffer = await archive.medias.fromDmMediaUrl(msg.mediaUrls[0], false, true) as ArrayBuffer;
// Write the file to disk
fs.writeFileSync('test_dir/my_img.jpg', Buffer.from(array_buffer));
```

## Continue

Next part is [Explore favorites](./Explore-favorites).

