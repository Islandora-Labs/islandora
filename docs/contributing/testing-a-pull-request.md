# Testing a Pull Request (i.e. Running Custom Code)

If you are testing a pull request, or for other reasons need to replace the
"official" code with code that's in a PR, or a different branch or fork, this
page offers three methods: using Composer Patches, using Composer to
require the branch and/or fork, or installing source repositories with Composer.

This documentation applies to Drupal modules, themes, and recipes, or any
other project that is managed by Composer.

!!! note "Can't I just `git clone`?"
    When managing a non-developmental site with Composer, it's very fragile to
    use commands like `git clone` to obtain code, as it can be overwritten by
    a `composer update`.

    For those who are comfortable cloning code onto a development environment, refer to
    [Installing Git repositories with Composer](#installing-git-repositories-with-composer).

!!! warning Changes to configuration
    If the change that you're testing involves a change to default config 
    that's shipped with a module, and the change does not update it during 
    an update hook, then you will need to load the applicable config
    separately. See "Updating configuration", below.

!!! note File ownership
    If possible, run the following commands as the user who owns the Drupal filesystem files.
    This may be `nginx` or `www-data` but is probably not `root`. If you're not able to, run 
    `chown -R [owning user] [affected files]` afterwards to reset the file ownership.jk

## Applying a Patch using Composer Patches

This method is best for testing pull requests, because it's very easy to get a
patch from a pull request. If the desired code is not the subject of a
PR, you can still use this method but must generate a `.patch` file yourself.
You may wish to use the [fork or branch](#using-composer-to-require-a-fork-or-branch)
method, as it is more dynamic.

Run the following commands from within your site's root folder. They will
update your `composer.json` file.

```shell
# Enable Composer Patches
composer config allow-plugins.cweagans/composer-patches true
composer require cweagans/composer-patches
```

For the next step, prepare the following replacement tokens:

* `MY_PACKAGE`: The full Composer name of the package to patch. It
  takes the form [vendor name]/[project name]. Example:
  `drupal/controlled_access_terms`
* `MY_ISSUE_TITLE`: A descriptive way to identify what the patch is for.
  Best practice is to include a link to the related issue -
  especially if you're going to keep this patch around for a while.
  Example: `Updated config format https://github.com/Islandora/controlled_access_terms/issues/117`
* `MY_PATCH_LOCATION`: Where to access the patch. See below.

The patch can be entered in composer.json as either a URL or as a reference
to a local file. To get the URL of a patch for a PR, go to the PR's main URL, and append
`.patch` to the URL. Make sure that your URL ends with `pull/XX.patch`
and not `pull/XX/files.patch` - the latter will not work.

If you don't have a PR, you could create a patch using `diff`. However, this
patch will be static and will need to be updated manually if your code
changes. You can put patches in a folder in your root directory such as `
[COMPOSER_ROOT]/assets/` and then the patch location would be
`assets/my_patch_name.patch`.

This one-liner, with the substitutions above, will add the patch to your
`composer.json`:

```shell
# Add patch to composer.json
composer config extra.patches --merge --json '{"MY_PACKAGE": {"MY_ISSUE_TITLE": "MY_PATCH_LOCATION"}}'

```

Or you could manually edit `composer.json` so it contains the following
(`...` denotes omitted content):
```json
{
  ...
  extra: {
    ...   
    patches: {
      "MY_PACKAGE": {
        "MY_ISSUE_TITLE": "MY_PATCH_LOCATION"
      }
    }
  }
}
```

Then, update your package (recall e.g. `drupal/controlled_access_terms`)
using Composer:

```shell
composer update MY_PACKAGE
```

The patch should apply, and then you will be running a patched version! If
you're using a dynamic patch, then running `composer update` again should
pull in changes to the code.

Finally, update your database and clear cache:

```shell
drush updb
drush cr
```

## Using Composer to require a fork or branch

This method is best if you don't have a pull request open for the code.

### Step 1: Add a repository (if necessary)

If your code is on a fork, then you will need to add a repository to
Composer so that it knows where to get your package.

If the code that you want to test is a different branch/tag on the same
repository that you're currently getting your code from, then you do not
need to add a repository.

Prepare the following replacement tokens before adding a repository:

* `REPO_NAME`: a name for this repository (mandatory if using the composer
  one-liner), e.g. `rosiel-islandora`
* `REPO_URL`: the URL to the repository, e.g. `https://github.com/rosiel/islandora`

```shell
# Add custom repo
composer config repo.REPO_NAME vcs REPO_URL
```
Your `composer.json` file should now contain

```json
{
  ...
  "repositories": {
    ...
    "REPO_NAME": {
      "type": "vcs",
      "url": "REPO_URL"
    }
  }
}
```

!!! note "Order of precedence of repositories"
    If you have a matching version spec (e.g. branch name) that's available
    from multiple repositories, e.g. both islandora's Gitlab and your personal
    fork both have an `enable-hocr` branch, then the repository that's first
    in the list in composer.json will take precedence.

### Step 2: Require the custom branch

This step could be as simple as
```shell
composer require MY_PACKAGE:dev-MY_BRANCH_NAME
```

with the following replacements:

* `MY_PACKAGE`: the full Composer name of the package. Example:
`drupal/islandora`
* `MY_BRANCH_NAME`: the name of the branch you want to run. Example:
  `testing-fedora-6`. Note that in the case that your branch name is
  "version-like" for example `2.x`, then the `dev` goes at the end, as in
  `2.x-dev`, instead of preceding the branch name as in the template above.


However, if your component is a dependency of another component, then you
will probably need to use an alias. This allows your custom code to "act as"
a version that will meet the requirements of your other component. For
example, if the `drupal/islandora_mirador` package requires
`drupal/islandora:^2.4.1`, then using Composer to require the `enable-hocr`
branch of `drupal/islandora` will not meet the requirements. Instead, use
`as` to provide an alias, to a version that will match the constraints. Note
the quotes around the package name and version spec. This takes the form:

```shell
composer require "MY_PACKAGE:dev-MY_BRANCH_NAME as ALIAS"
```
For example:
```shell
composer require "drupal/islandora:dev-enable-hocr as 2.12.1"
```

That will install the specified branch and allow it to work with your
dependencies.

## Installing Git repositories with Composer

This method pulls the source (i.e. Git) repositories directly into your project
and will allow you to pull in open pull requests by simply following a normal
Git workflow.

!!!note
    If the site has already been installed without using `--prefer-source` you
    will need to clear Composer's cache via `composer clearcache` or including
    `--no-cache` for any `install` or `reinstall` commands (as below).

### Step 1: Re-install the code from source (if required)
Assuming that the environment has not been installed with `--prefer-source`,
reinstall the package.

```shell
composer reinstall MY_PACKAGE --prefer-source
```
with the following replacements:

* `MY_PACKAGE`: the full Composer name of the package. Example:
  `drupal/islandora`

This will pull the code from the source repository (i.e. including the `.git`
files) and add the package code at the same version of the `MY_PACKAGE`
that was previously installed and is in the lock file.

For example:
```shell
composer reinstall "drupal/islandora" --prefer-source --no-cache
```

### Step 2: Pull the code from the pull request to review.
Follow Github's [documentation](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/checking-out-pull-requests-locally)
to pull the code from the pull request to review.

Now you can test your specified code, as well as edit the code and create
commits. Note that if you're doing this in a throwaway environment such as a VM
or a Docker Container, you will need to configure authentication (e.g.
install an SSH key with Github) before you can push your commits.

Finally, update your database and clear cache:

```shell
drush updb
drush cr
```

## To reset these changes

### ... using Composer Patches

When you no longer need to be applying the patch, simply remove it from the
`patches:` section of `composer.json` (and of course, take care to ensure the
json remains valid, by adjusting commas!) and run `composer update MY_PACKAGE`.

### ... using Composer require

When you no longer want to pull from a separate branch or fork, reset the
version constraint back to what it used to be, or, if your package was not
originally in `composer.json` (because it is required as a dependency by
another package), you can delete the requirement from your `composer.json`.
Then run `composer update MY_PACKAGE`. If you added a repository, it's
safest to delete the repository, as it could lead to you getting stale
branches from a fork rather than the desired active code from the canonical
repository.

### ... using source repositories

When you no longer want the custom code present simply reset the branch back to
the default branch or tag.

More great information is available in the [Composer Documentation](https://getcomposer.org/doc/).

Note that when resetting changes to code, this will not reset changes made to your configuration.
If your configuration was changed, consider setting it back.

## Updating Configuration

The above steps make code available to Drupal. If there is an additional configuration change that's
needed for what you're testing and it's not addressed by an update hook, then 
you may need to install the configuration manually.

Note that these changes to configurations may override settings that you've deliberately set, so please
back up your original configuration before proceeding! Also, there is no automated way to set configuration
"back" to what it was, so ensure that you save a copy locally.

### With Configuration Import/Export

You can use Drupal's built in configuration import/export feature to export (save a copy of) your original configuration
and to import the new configuration. This is available at Manage > Configuration > 
Development > Configuration Synchronization > Export / Import.

### With Devel's Config Editor

A slightly faster method is to use the Devel module's Config Editor. It appears (if configured to) on the Devel
toolbar. Here you can see the current configuration, save a copy, and enter the new desired configuration all 
in one place.

### With Drush

Drush does not let you import a single config file, but you could export your entire site's configuration, add the new
file to it, and re-import the entire site's configuration.

```shell
drush config:export -y
cp [PATH_TO_FILE_ABOUT_TO_BE_OVERRIDDEN] ~ [or your desired location]
cp [PATH_TO_CONFIG_FILE] config/sync/ 
drush config:import -y
```

You may need to adjust file permissions (see note at top) if you're not acting as the owner of the Drupal files.
