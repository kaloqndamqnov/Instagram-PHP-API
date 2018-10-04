# ![Image](example/assets/instagram.png) Instagram PHP API V2

> **Note:** Starting today (January 30), we are launching three new features on the Instagram Graph API designed to help businesses better manage their organic presence on Instagram. As part of our API changes, we will also be deprecating the older Instagram API Platform over the next two years beginning on July 31, 2018.Please see the full summary [here](https://developers.facebook.com/blog/post/2018/01/30/instagram-graph-api-updates/).

> **Note:** On the 17 Nov 2015 [Instagram](http://developers.instagram.coThe following endpoints are deprecated immediately:m/post/133424514006/instagram-platform-update) made [changes to their API ](https://instagram.com/developer/changelog/). Apps created before Nov 17, 2015 wont be affected until Jun 2016. Apps created on or after Nov 17 2015 will require to use their updated API. Please note that this library doesn't yet support their new updates. For more information, please see [#182](https://github.com/cosenary/Instagram-PHP-API/issues/182).

A PHP wrapper for the Instagram API. Feedback or bug reports are appreciated.

[![Total Downloads](http://img.shields.io/packagist/dm/cosenary/instagram.svg?style=flat)](https://packagist.org/packages/cosenary/instagram)
[![Latest Stable Version](http://img.shields.io/packagist/v/cosenary/instagram.svg?style=flat)](https://packagist.org/packages/cosenary/instagram)
[![License](https://img.shields.io/packagist/l/cosenary/instagram.svg?style=flat)](https://packagist.org/packages/cosenary/instagram)

> [Composer](#installation) package available.  
> Supports [Instagram Video](#instagram-videos) and [Signed Header](#signed-header).

## Requirements

- PHP 5.3 or higher
- cURL
- Registered Instagram App

## Get started

To use the Instagram API you have to register yourself as a developer at the [Instagram Developer Platform](http://instagr.am/developer/register/) and create an application. Take a look at the [uri guidelines](#samples-for-redirect-urls) before registering a redirect URI. You will receive your `client_id` and `client_secret`.

---

Please note that Instagram mainly refers to »Clients« instead of »Apps«. So »Client ID« and »Client Secret« are the same as »App Key« and »App Secret«.

---

> A good place to get started is the [example project](example/README.md).

### Installation

I strongly advice using [Composer](https://getcomposer.org) to keep updates as smooth as possible.

```
$ composer require propeopleua/instagram
```

### Initialize the class

```php
use MetzWeb\Instagram\Instagram;

$instagram = new Instagram(array(
	'apiKey'      => 'YOUR_APP_KEY',
	'apiSecret'   => 'YOUR_APP_SECRET',
	'apiCallback' => 'YOUR_APP_CALLBACK'
));

echo "<a href='{$instagram->getLoginUrl()}'>Login with Instagram</a>";
```

### Authenticate user (OAuth2)

```php
// grab OAuth callback code
$code = $_GET['code'];
$data = $instagram->getOAuthToken($code);

echo 'Your username is: ' . $data->user->username;
```

**All methods return the API data `json_decode()` - so you can directly access the data.**

## Available methods

### Setup Instagram

`new Instagram(<array>/<string>);`

`array` if you want to authenticate a user and access its data:

```php
new Instagram(array(
	'apiKey'      => 'YOUR_APP_KEY',
	'apiSecret'   => 'YOUR_APP_SECRET',
	'apiCallback' => 'YOUR_APP_CALLBACK'
));
```

`string` if you *only* want to access public data:

```php
new Instagram('YOUR_APP_KEY');
```

### Get login URL

`getLoginUrl(<array>)`

```php
getLoginUrl(array(
	'basic',
	'likes'
));
```

**Optional scope parameters:**

<table>
	<tr>
		<th>Scope</th>
		<th>Legend</th>
		<th>Methods</th>
	</tr>
	<tr>
		<td><code>basic</code></td>
		<td>to use all user related methods [default]</td>
		<td><code>getUser()</code>, <code>getUserFeed()</code>, <code>getUserFollower()</code> etc.</td>
	</tr>
	<tr>
		<td><code>relationships</code></td>
		<td>to follow and unfollow users</td>
		<td><code>modifyRelationship()</code></td>
	</tr>
	<tr>
		<td><code>likes</code></td>
		<td>to like and unlike items</td>
		<td><code>getMediaLikes()</code>, <code>likeMedia()</code>, <code>deleteLikedMedia()</code></td>
	</tr>
	<tr>
		<td><code>comments</code></td>
		<td>to create or delete comments</td>
		<td><code>getMediaComments()</code>, <code>addMediaComment()</code>, <code>deleteMediaComment()</code></td>
	</tr>
</table>

### Get OAuth token

`getOAuthToken($code, <true>/<false>)`

`true` : Returns only the OAuth token  
`false` *[default]* : Returns OAuth token and profile data of the authenticated user

### Set / Get access token

- Set the access token, for further method calls: `setAccessToken($token)`
- Get the access token, if you want to store it for later usage: `getAccessToken()`

### User methods

**Public methods**

- `getUser($id)`
- `searchUser()`
- `getUserMedia($limit = 0, $min_id = null, $max_id = null)`

**Authenticated methods**

- `getUser()`
- `getUserMedia( <$limit>)`

> [Sample responses of the User Endpoints.](http://instagram.com/developer/endpoints/users/)

---

### Tag methods

**Public methods**

- `getTag($name)`
- `getTagMedia($name)`
- `searchTags($name)`

> [Sample responses of the Tag Endpoints.](http://instagram.com/developer/endpoints/tags/)

## Instagram videos

Instagram entries are marked with a `type` attribute (`image` or `video`), that allows you to identify videos.

An example of how to embed Instagram videos by using [Video.js](http://www.videojs.com), can be found in the `/example` folder.

---

**Please note:** Instagram currently doesn't allow to filter videos.

---

## Signed Header

In order to prevent that your access tokens gets stolen, Instagram recommends to sign your requests with a hash of your API secret, the called endpoint and parameters.

1. Activate ["Enforce Signed Header"](http://instagram.com/developer/clients/manage/) in your Instagram client settings.
2. Enable the signed-header in your Instagram class:

```php
$instagram->setSignedHeader(true);
```

3. You are good to go! Now, all your requests will be secured with a signed header.

Go into more detail about how it works in the [Instagram API Docs](http://instagram.com/developer/restrict-api-requests/#enforce-signed-header).

## Pagination

Each endpoint has a maximum range of results, so increasing the `limit` parameter above the limit won't help (e.g. `getUserMedia()` has a limit of 90).

That's the point where the "pagination" feature comes into play.
Simply pass an object into the `pagination()` method and receive your next dataset:

```php
$photos = $instagram->getTagMedia('kitten');

$result = $instagram->pagination($photos);
```

Iteration with `do-while` loop.

## Samples for redirect URLs

<table>
	<tr>
		<th>Registered Redirect URI</th>
		<th>Redirect URI sent to /authorize</th>
		<th>Valid?</th>
	</tr>
	<tr>
		<td>http://yourcallback.com/</td>
		<td>http://yourcallback.com/</td>
		<td>yes</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/</td>
		<td>http://yourcallback.com/?this=that</td>
		<td>yes</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/?this=that</td>
		<td>http://yourcallback.com/</td>
		<td>no</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/?this=that</td>
		<td>http://yourcallback.com/?this=that&another=true</td>
		<td>yes</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/?this=that</td>
		<td>http://yourcallback.com/?another=true&this=that</td>
		<td>no</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/callback</td>
		<td>http://yourcallback.com/</td>
		<td>no</td>
	</tr>
	<tr>
		<td>http://yourcallback.com/callback</td>
		<td>http://yourcallback.com/callback/?type=mobile</td>
		<td>yes</td>
	</tr>
</table>

> If you need further information about an endpoint, take a look at the [Instagram API docs](http://instagram.com/developer/authentication/).

## Example App

![Image](http://cl.ly/image/221T1g3w3u2J/preview.png)

This example project, located in the `example/` folder, helps you to get started.
The code is well documented and takes you through all required steps of the OAuth2 process.
Credit for the awesome Instagram icons goes to [Ricardo de Zoete Pro](http://dribbble.com/RZDESIGN).

## Changelog

Please see the [changelog file](CHANGELOG.md) for more information.

## Credits

Copyright (c) 2011-2015 - Programmed by Christian Metz

Released under the [BSD License](LICENSE).
