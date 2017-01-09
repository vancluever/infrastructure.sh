# infrastructure.sh

This is a bash script that helps you control infrastructure-related jobs that
run [Terraform][1] in your CI system to deploy your infrastructure.

[1]: https://terraform.io/

## What You Need

 * A repository where your Terraform code resides in `terraform/`.
 * Your versioning scheme must be [semantic versioned][2] with the versions
   being written to tags.
 * You do most, if not all, of your work out of `master` with your remote
   configured for `origin`.
 * You have the following installed:
  * `git`
  * `bash` 4+
  * And you are on a system that has `cat`.

[2]: http://semver.org/

## The Concept

Your code resides in a repository along with your Terraform code. This code is
versioned and released using a versioning scheme that matches something like
what would be managed by [`release.sh`][2].

[3]: https://github.com/vancluever/release.sh

This script will then search the repository for all tags that match the `vX.X.X`
semver-style versioning format. The last 10 are presented to you, from which you
can choose a version, either the most recent one (the default), or a previous
one (in the event you need to roll back).

This script writes out a `terraform/version.tf` file that contains the
`build_version` variable. You can then handle this variable any way in your
Terraform code that you need to.

After writing out this version, the file is committed with a computed tag in the
message. This will be the tag that this commit gets tagged with after. The tag
follows the format:

```
infrastructure#BUILDNUM-VERSION
```

The build number is unique and is attached as a note to this commit as a Git
note under the `infrastructure_build_number` reference. This is incremented
every time a new run of this script happens.

As mentioned, after the commit happens, this is the tag that gets written out.
After the commit, tag, and new build number note are written out, everything
gets pushed.

## What do I do with the Tag?

In your CI system (ie: Travis), set up your build rules to run your Terraform
operations off of tags that start with `infrastructure`. Example for Travis can
be found in [this project][4].

[4]: https://github.com/vancluever/ubc_demo

The tag is designed to be verbose enough that it's easy to pick out in your CI
system.

## Re-running a Failed Terraform Run

If your Terraform run fails, you have two options:

 * For **non-source code or TF config issues**, you can correct the problems
   that need to be corrected in your CI system or elsewhere (example: correct
   missing or misconfigured variables, fix AWS credentials, etc), and just
   re-run the job if that's available to you.
 * For issues with the TF code, correct the changes, commit them, and re-run the
   script.
    * The script can be re-run even if you don't need to change any code, but
      would like the build number incremented. `version.tf` is timestamped, so
      there will always be something to commit.
    * Note that this script **only** commits `terraform/version.tf`. If you need
      to correct other parts of the TF code make sure you commit your changes
      before re-running the script.

## License

```
This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <http://unlicense.org/>
```
