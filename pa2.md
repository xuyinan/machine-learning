# ￼Assignment 2: Authentication and Sessions
### Due Tuesday, Oct 6, 2014, at 11:55 PM.

For this PA, you will continue working on the photo album website developed in PA1. However, do not touch the files in the pa1 sub-directory. Make another sub-directory called pa2, and copy the files from pa1 into the pa2 sub-directory and work on the files there. **Modifying your pa1 folder after the project deadline is in violation of the honor code.** Also, it can hurt the grading process. By the end of this programming assignment you will learn how to authenticate users and maintain sessions. **Also, we are using new databases for this project, databases name: groupxxpa2.**

 Finally, make sure to change your routes to '{secret-key}/**pa2**/'.

## Part 1: Getting started

These sites contain useful tutorial and reference information for what you'll be implementing in this PA.

* [HTTP Cookies](http://en.wikipedia.org/wiki/HTTP_cookie)
* Sessions: [Python](http://flask.pocoo.org/docs/quickstart/#sessions)
* Authentication: [Python](http://flask.pocoo.org/snippets/8/)

## Part 2: Build the website
This PA is about personalization. The first step to doing any kind of
personalization is to keep track of who is browsing your site. In
class we discussed how HTTP is a stateless protocol, which cannot
itself retain data from one request to the next.  The way to maintain
state from one page request to another is using *sessions*. In this PA
we will add a login page to the site. Users will only need to type
their username and password once. Thereafter, your website will use
session variables to determine who the logged in user is.

Pages that are *sensitive* require users to login before they can view those pages. The rest of the pages will remain public and will not require a username or password to be viewed. Whenever a user tries to enter a sensitive page you should make sure that he/she has the privileges to view it. This is done by checking if the user has a valid session, and if so, whether the user is authorized to view the page.

Some pages do not require user authentication or sessions (e.g., a
default home page or a create account page). Some other pages only
require that the user be authenticated and are not dependent on who
the user is (e.g., a logged in home page). Others may provide access
depending on who the user is and whether he/she is permitted to access
that page (e.g., someone's private album).

In short, when a user requests a URL, you should:
* Check if the page is public. If so, view the page.
* Check if the page is sensitive and the user has a valid session. If so, check if the
user has permission to see the page, and if so, let them see the page.
* If the page is sensitive and the session has expired, say so and give the user a link to the Login page.￼￼￼￼
* If the page is sensitive and there is no session, then state that one must authenticate to view this page and give a link to the Login page.

In some of the cases above, the user is redirected to the Login page. This Login page needs to remember what the requested sensitive page was using a query parameter. For example: `/login?url=/the/prev/url` After the user types her name and password she should be returned right back to the sensitive page she previously tried to access via a redirect again. Note, some of the pages can be sensitive some of the time and public the rest of the time. For example, the View Album page is only sensitive if the album is private.

**Your code should observe the following rules about access privileges:**
* Public albums are _accessible_ to both logged in users and unauthenticated visitors.
* Private albums are _accessible_ only to those users that have explicit access to that
album. Users will have access to other user's private album if
and only if there exists a tuple (a, u) in the *AlbumAccess* relation
(see below).
* A picture that does not belong to any public albums is accessible to a user if and only if it belongs to at least one private album that _u_ has permission to access to.*

For example, User X creates albums A (public), B (private), and C (private). X grants User Y access to B. Y creates album D.

Y can view albums A, B, and D on his logged-in index page. These are the albums Y has permission to see.

Y can *only* view album D on his logged-in '/albums' page, by clicking on the link to 'My Albums'. These are the albums Y has permission to *edit*

X has permission to view the albums A, B, and C, both on his logged-in index page, and '/albums' page. He can edit these albums.



### Add public/private feature to the albums

#### Update Album Table

You need to add `access` attribute to this table, so the new scheme for Album will be
* Album ( *albumid*, title, created, lastupdated, username, access )

`access` specifies whether access to the album should be limited to a
set of users indicated in the *AlbumAccess* table (described below).
It only takes values of "public" or "private." It should be of type `varchar`.

#### Create new Table AlbumAccess

* AlbumAccess ( *albumid*, *username* )

This relation indicates the users who have access to each specific album.

In `/album/edit`, the user should be able to edit the `access` permission for the
*Album* table. The user should also be able to give/remove user's access by editing the AlbumAccess table.Thus an album could be public, private or private with someone accessible. Only the album's owner should be able to modify the album users' access permissions. If the album is not public, on `/album/edit`, you should have a table **that is only displayed for the owner** where the album's owner can view permissions of everyone with access to the album and type in a username to grant permissions. Public albums should not display this table.

The interface for `/album/edit` should appear roughly as below:

<table>
<tr><td>Username</td><td>Update Access</td></tr>
<tr><td>sportslover</td><td>[Revoke]</td>
<tr><td>traveler</td><td>[Revoke]</td></tr>
<tr><td>New: ______________</td><td>[Add]</td></tr>
</table>

The interface for `/albums/edit` should appear roughly as below:



<table>
<tr><td>Album</td><td>Access</td><td></td><td></td></tr>
<tr><td>Summer 2011 in Iceland</td><td>Public</td><td>[Edit]</td><td>[Delete]</td></tr>
<tr><td>Spring break 2010 in Brooklyn</td><td>Public</td><td>[Edit]</td><td>[Delete]</td></tr>
<tr><td>Thanksgiving 2010</td><td>Public</td><td>[Edit]</td><td>[Delete]</td></tr>
<tr><td>New: ______________</td><td>________</td><td>[Add]</td><td></td></tr>
</table>

The owner of an album should *not* appear in AlbumAccess for that album

### Final Schema
The Final Schema should be the *same* as PA1, except for the following additions:

1) Update Album table to have attribute 'access'

2) Add AlbumAccess table, as described above

3) IF doing Extra Credit with Root user, User table should reflect this

4) IF doing Extra Credit with user confirmation, User table should reflect this

Just like PA1, you need a sql folder with the files 'tbl_create.sql' and 'load_data.sql'. You can/should use your PA1 files as a starting point.

*****

### Session Management

Sessions allow us to reliably keep track of User information while the User
is browsing our website. *This project is centered around the use of sessions.*

A good, concise explanation on sessions: http://stackoverflow.com/questions/3804209/what-are-sessions-how-do-they-work

You should enforce a session inactivity time limit of 5 minutes. If a user tries to continue a session after 5 minutes or more of inactivity, then log out the user, destroy the session, and force the user to log in again.

You may want to maintain two session variables: `username` and `lastactivity`. The `username` stores the user name of the authenticated user. The `lastactivity` stores the time of user’s last activity to check inactivity time out.

##### Sessions in Python:
In Flask sessions are started automatically. The session variables can be accessed as such: `session['username']`.
In order for sessions to work properly however a secret key must be set. Please refer to the Flask docs for more infomation about how to use sessions. Be sure that sessions are imported when attempting to use them. Sessions work similarly in other Python frameworks.

The User's session is only maintained and used on the server-side; There is no need to write client-side code to maintain sessions. For simplicity in this project, assume cookies are *always enabled*.

### Implement Sessions and Authentication: 

What follows is a list of the files that you should create in your
application. You should have created some of these for PA1.
*page is either in a public state (no session involving) or sensitive state (need session to manage page).*

#### Default home page:`/` [public]

Contains welcoming message and information about the website. Also has an invitation for new users to join as members. There should be a link to a New User page. There should be some way of getting from this home page to all the public albums of all users.

#### New User page:`/user` [public]

Contains a form for users to fill in their username, firstname,
lastname, e-mail address and password.  Make sure the password field
does not display text and that there are two password fields for
verification.

There are validation rules set forth in Part 4 below that describe the
set of permissible passwords.  You should use client-side HTML5 validation
checking to ensure that the password is "good enough."  You should
*not* use JavaScript to perform this testing.

You should also check that the two password fields match.  You should
perform this test at both the client- and server-side.  For this test *only*
you can use a small JavaScript function.

Python is a server-side scripting languages which means it is executed by the web server when the user requests a document. JavaScript is mainly a client-side scripting language, normally embedded within HTML, and it will be executed on client. This will prevent an unnecessary round-trip if the user fills in two unmatching passwords. Why check server-side, then? Because if Javascript is turned off, we still need to be able to check if the two passwords match.

If a User provides all valid information, create the User by inserting their information into your database. Please be sure to encrypt their password! 

If a session already exists redirect the user to `/user/edit`.

#### Logged in home page:`/` [sensitive]

This page is the home page for a user who has already logged in. Make
sure for all pages in a logged in state, you clearly display the
message "Logged in as <Firstname> <Lastname>". This page and all
subsequent logged-in pages should have a navigational interface with
links to Home (this page), Edit Account, My Albums and Logout. The
main body of the page should have a list of all the accessible albums
categorized by their owners. Accessible albums include public albums
as well as private albums which have been authorized for the current
logged in user by the owner. 

#### Edit Account page:`/user/edit` [sensitive]

The user should be able to change his/her firstname, lastname,
password and e-mail address (but not username). Again, validate the input values
both on the client-side as well as server-side. Also keep a link on this
page to delete the user's account. This should remove all associated
pictures , albums, as well as access to other users' albums (useful in case the username
is recycled and allocated to a new user). 

You can perform the delete however you like. However creating a `/user/delete` endpoint which 
accepts only POST requests may be the easiest, then you can redirect the user back to the 
public homepage.

### User Login page:`/user/login` [public]
Here, a non-authenticated user can enter their username and password to login to a specific user
account and become authenticated. Refer to Part 3 "Authentication" and Part 4 "Validation" 
for more details on successfully logging in users, and how to notify users when they incorrectly 
attempt a login.

### My Albums page:`/albums` [sensitive]
This page is similar to its corresponding route in PA1, in that it contains links to albums which 
the current User owns, as well as a link to the ‘/albums/edit’ route. Note that, 
instead of using a URL parameter to input the username of the User’s albums that 
we want to see, we are instead using the current session. You can navigate here 
by clicking on the ‘My Albums’ button at the index page.

### Public Albums of All Users:`/albums` [public]
This page shows all of the public albums to a User who isn’t logged in. 
Here, you find links to view the albums, but no links to edit them. 
You can navigate here from the index page.

#### My Albums page:`/albums/edit` [sensitive]

This is your `/albums/edit` page from PA1. It allows the user to
add new albums, view existing albums, delete them or edit them.
Remember that deleting an album should also involve deleting
pictures in the album. *Note that you can only see those albums you created*

Note that all new albums have private access by default.

#### Edit Album page:`/album/edit` [sensitive]
￼￼￼
At the top of this page the user should be able to change the album
name and permissions. There should be some way the user can edit a
list of other users to whom he/she would like to give explicit access
to view this album, if it is private. If an Album become public, *all
users that don't own the album but have permission to view it should
lose said permissions.* You should also list the  pictures in the album. 
Users should be able to delete pictures from the album as well as add new pictures.
Users should also be able to click on individual images and be directed to `/pic` 
from your PA1. Make sure to keep the Album.lastupdated field in the database
updated when you change title, caption, or permission for an album, as well as
all album updates from PA1. This implies that pic captions are editable (as described in '/pic').

#### View Album page:`/album` [sensitive/public]

This page displays the thumbnail view of an album just like the previous assignments. 
The album title should be at the top, along with the album's owner. The
photos should be displayed in sequence order, each with its date, and
a caption (if caption exists for that photo). The page is the same as in the previous assignment, except
that if the album is private, only the logged-in user has permission
to view the album. This means the `/album` can be reached
either from the logged-in user’s homepage or your albums page for
non-logged in users (assuming correct permissions).

#### View picture page:`/pic` [sensitive/public]

This page displays a picture just like the previous assignment. It should 
have the caption, full-sized picture and links to previous and next picture.
You must be able to edit the caption. If you do, make sure to update the Album.lastupdated
field for the album that pic is in. If the user does not have access to the album 
this picture is in, they should not be able to see the picture.

#### Logout page:`/logout` [sensitive]

This should destroy the session and redirect to the default home page.

## Part 3: Authentication

Perhaps obviously, authentication issues are present throughout this assignment. 
In order to view a sensitive page, the user must be authenticated. You should use 
a form with a username/password that is shipped to a server-side script that 
tests the information against the database. 
Login error should yield one of two situations, each of which should be handled differently.

* Username is invalid: Complain that the username does not exist.
* Username-password combo is invalid: Complain that Username-password combo is invalid.

A complaint can be as simple as a sentence notifying the user of his incorrect attempt. 
However, *it must be specific to the error*.

## Part 4: Validation

The new HTML5 specification standardizes many features that are
previously common across the web, but were previously implemented in a
piecemeal and/or quirky fashion.  One such example is client-side
validation of forms.  Previously implemented using JavaScript, you can
now implement client-side form validation with HTML5's built-in
support.  (See
http://www.html5rocks.com/en/tutorials/forms/html5forms/#toc-validation)

Again: you should *not* use JavaScript for form validation, except to test
that multiple fields contain the same value.

You should enforce the following rules:

* The username must be unique (this can only be enforced server-side)
* The username must be at least three characters long
* The username can only have letters, digits and underscores
* The password should be at least 5 and at most 15 characters long
* The password must contain at least one digit and at least one letter
* The password can only have letters, digits and underscores
* E-mail address should be syntactially valid (i.e., it *could* be a valid email address, whether or not it is actually someone's working E-mail address)
* Except for password, all fields (username, firstname, lastname, and e-mail address) have a max length of 20

You can assume that the user is acting in good faith: your goal is to
prevent users from adding bad usernames/passwords, not to guard
against motivated attackers who want to sneak a [strange entry](http://en.wikipedia.org/wiki/Code_injection) into your password database. (which means you do not need to check things beyond above rules)

As mentioned in above sections, server-side validation of matching passwords (the two passwords the user entered when signing up/editing their information) is necessary. In the event that the two passwords don't match on the server-side, redirect to the page that the information was entered on with an error about User sign-up failing (it can be generic). Make sure to also do this same event (i.e., redirect
    back here and display error) if the username already exists. 

## Grading

We will check the pa2 directory for your new secure photo album website. Based on PA1, your website should contain at least the following users with their albums.

* Username: sportslover, Password: paulpass93 - "I love football" (public), "I love sports" (public)
* Username: traveler, Password: rebeccapass15 - "Around The World"(public)
* Username: spacejunkie, Password: bob1pass - "Cool Space Shots" (private, also
accessible to traveler)

Your website may contain other users and albums, but please ensure
that the above users and albums exist. Do not touch the files in pa2
sub-directory after the deadline.

As mentioned before, **Remember to commit your code into GitHub and the server, 
please do not modify your code after the due date - either on the repo or the 
server**, or else we will assume your submission is late. We then can assess 
late days or take off points.

## Extra Credit

Each correctly implemented extra credit will increase your score by 2%, to a maximum of 6%. The main reason for the extra credit is to provide opportunities for you to challenge yourself. Please take on the extra credit after you have completed the rest of the assignment. Make sure to mention which extra credit features you implemented in your README.md.

* When form is submitted in the New User page, send an e-mail message to 
the new user confirming his/her membership and redirect them to the logged in 
home page. (HINT: Check out flask-mail - Ask Josh for help.)
* Ask the user if he/she has forgotten the password and if so, create a 
new password and e-mail it to them.
* An additional class of "root-user". Anyone who is a root-user can edit anyone's album.
There should be an administrative interface for giving/removing the root-user privilege.
* Use CSS to align images in rows and columns (no HTML tables allowed!) and make 
the website look more appealing - GSI/IAs will award points based on effort and overall design.
