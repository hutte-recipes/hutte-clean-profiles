# Hutte Recipe - Remove Permission Assignments from Profiles

> This recipe provides a Custom Button in Hutte to remove permissions from all Profiles in the force-app directory. Under the hood this uses Perl with Regular Expressions.

The end of life of permissions on profiles has been announced by Salesforce (see https://admin.salesforce.com/blog/2023/permissions-updates-learn-moar-spring-23), and it is time to move all your permission management to permission sets. To avoid that redundant and outdated profile based permissions "pollute" your Git based source, we suggest to clean your profiles from permission assignments, every time you touch upon those in your work.

## Prerequisites

- a valid Sfdx Project
- a `hutte.yml` file (e.g. the default one shown in the `CONFIGURATION` tab)

## Step 1: Add custom scripts

- Edit the `hutte.yml` file in your default branch
- Add the following lines to the `custom_scripts`
- Push it to your main branch

```yaml
custom_scripts:
  # This scripts will be displayed on the scratch org's page
  scratch_org:
    "Clean Profiles":
      description: Remove Permission Assignments from all Profiles in force-app
      run: |
        #!/usr/bin/env bash
        set -eo pipefail
        apk add perl

        PERL_CODE="$(cat <<END
        my @elementsToRemove = ('classAccesses', 'fieldPermissions', 'objectPermissions', 'pageAccesses', 'tabVisibilities', 'userPermissions');
        for my \$elementToRemove (@elementsToRemove) {
          s|(\s*<\$elementToRemove>.*?</\$elementToRemove>)||gse
        }
        END
        )"

        find force-app -name "*.profile-meta.xml" -exec perl -0777 -p -i -e "$PERL_CODE" {} +
        git add .
        git commit -m "Remove permission assignments from all Profiles included in your project"
        git push
```

\*Note: This custom button can also be added to sandboxes page in Hutte, for that, add the button to `sandbox` instead of `scratch_org`

## Step 2: Execute

- Create a Scratch Org
- Optionally "Pull Changes" with modified Profiles
- Use the `Clean Profiles` custom button to remove Permission Assignments from all Profiles in your project
