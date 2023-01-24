# Smart impact test (VUE.JS)

## Development start:

npm install
complete the .env template file with the values received.
npm run serve
go to : http://localhost:8081/

## Build start:

npm install --global serve
run in project root: cd dist && serve

### Build does not require .env!

### Info about the project:

This project uses a text input to search an user and it's gists using the github api.
The input has 3 validations:
-Required
-Don't begin with "-"
-Smaller than 39 characters
If an user is found, the informations will be presented andthe gists are loaded, otherwise, a message is presented.
If the request fails, a message is presented.

The forks are loaded on a gist file click for faster initial loading.
On file click, the code is loaded & highlighted.

For a better memory management, only the gists & user data is stored as variables. Code blocks & forks are loaded on click and inserted directly into html using vanilla javascript.

The project is responsive, but the code blocks (the library breaks the responsiveness)

In the next steps, a better design should be added and code blocks colors should be fixed.
The user and the gists could be optimized and inserted also directly into document.
