# Github and Gitlab Integration

## Overview
This document provides guidelines on how to pull repositories from Github in Gitlab.

## Integration Guidelines

* Open you Github profile and move to this location `Settings -> Developer Setting -> Personal Access Token`.

* Now click the `Generate new token` button to create a new token.

* Provide a brief description about the purpose of the token.

* Select the scopes, or permissions, you'd like to grant this token. To use your token to access repositories from the command line, select `repo` checkbox.

* Once configurations are done, generate the token.

## How to use the Personal Access Token (PAT)

* Once you have a token, you can enter it instead of your password when performing Git operations over HTTPS.

* COPY your repository HTTPS link

    ```bash
    $ https://github.com/<organization>/<repo>.git
    ```

* Add your token at the location specified below:
    ```bash
    $ https://<add-token-here>@github.com/<organization>/<repo>.git
    ```
* Now repository can be pulled with using username and password.

## Notes

* As a security precaution, GitHub automatically removes personal access tokens that haven't been used in a year.

## Refrences

* https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line