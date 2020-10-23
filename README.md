# PDK Templates

The PDK Templates is the default templates repository for use with the [Puppet Development Kit](https://github.com/puppetlabs/pdk), within which we have defined all the templates for the creation and configuration of a module. Look into these directories to find the templates:

* `moduleroot` templates get deployed on `new module`, `convert` and `update`; use them to enforce a common boilerplate for central files.
* `moduleroot_init` templates get only deployed when the target module does not yet exist; use them to provide skeletons for files the developer needs to modify heavily.
* `object_templates` templates are used by the various `new ...` commands for classes, defined types, etc.

PDK builds the configuration of the module by reading a set of default configuration from `config_defaults.yml` and merging it with the contents of `.sync.yml` from module's root directory, if exists. Top-level keys of the resulting hash correspond to target files or represent a global configuration. Global configuration will be merged with the configuration hash for a particular target file. This allows a module developer to override/amend the configuration by putting new values into `.sync.yml`. A knockout prefix may be applied to elements within `.sync.yml` to [remove elements from the default set](#removing-default-configuration-values). Target files are created by rendering a corresponding template which is referring its configuration via the `@configs` hash. The data for a target file may include `delete: true` or `unmanaged: true` in order to have the particular file removed from the module or left unmanaged, respectively.

* [Basic usage](#basic-usage)
* [Values of config\_defaults](#values)
* [Making local changes to the Template](#making-local-changes-to-the-template)
* [Further Notes](#notes)

## Basic Usage

Templates like this one can be used in conjunction with the PDK. As default the PDK itself uses the templates within this repository to render files for use within a module. Templates can be passed to the PDK as a flag for several of the commands.

> pdk convert --template-url https://github.com/puppetlabs/pdk-templates

Please note that the template only needs to be passed in once if you wish to change it, every command run on the PDK will use the last specified template.
For more on basic usage and more detailed description of the PDK in action please refer to the [PDK documentation](https://github.com/puppetlabs/pdk/blob/master/README.md).

## Values of config\_defaults <a name="values"></a>

The following is a description and explaination of each of the keys within config\_defaults. This will help clarify the default settings we choose to apply to pdk modules.

### common

> A namespace for settings that affect multiple templates

| Key                    | Description |
|:-----------------------|:------------|
| disable\_legacy\_facts | Set to `true` to configure PDK to prevent the use of [legacy Facter facts][legacy_facts_doc]. Currently this will install and enable the [legacy\_facts][legacy_facts_pl_plugin] plugin for puppet-lint for use during `pdk validate`. |


### .gitattributes

>A .gitattributes file in your repo allows you to ensure consistent git settings.

| Key               | Description   |
| :-----------------|:--------------|
| include           | Defines which extensions are handled by git automatic conversions (see the [gitattributes](https://git-scm.com/docs/gitattributes) documentation). The default configuration helps to keep line endings consistent between windows and linux users.|

### .gitignore

>A .gitignore file in your repo allows you to specify intentionally untracked files to ignore.

| Key               | Description   |
| :-----------------|:--------------|
| required          | The default list of files or paths for git to ignore or untrack that are commonly specified in a module project.
| paths             | Defines any additional files or paths for git to ignore or untrack. (see the [gitignore](https://git-scm.com/docs/gitignore) documentation).

### .pdkignore

>A .pdkignore file in your repo allows you to specify files to ignore when building a module package with `pdk build`.

| Key               | Description   |
| :-----------------|:--------------|
| required          | The default list of files or paths for PDK to ignore when building a module package.
| paths             | Defines additional files or paths for PDK to ignore when building a module package.

### Rakefile

>[Rake](https://github.com/ruby/rake) is a Make-like program implemented in Ruby. Tasks and dependencies are specified in standard Ruby syntax within the Rakefile, present in the root directory of the code repository. Within modules context Rake tasks are used quite frequently, from ensuring the integrity of a module, running validation and tests, to tasks for releasing modules.

| Key            | Description   |
| :------------- |:--------------|
|requires|A list of hashes with the library to `'require'`, and an optional `'conditional'`.|
|changelog\_user|Sets the github user for the change_log_generator rake task. Optional, if not set it will read the `author` from the `metadata.json` file.|
|changelog\_project|Sets the github project name for the change\_log\_generator rake task. Optional, if not set it will parse the `source` from the `metadata.json` file|
|changelog\_since\_tag|Sets the github since_tag for the change\_log\_generator rake task. Required for the changlog rake task.|
|changelog\_version\_tag\_pattern|Template how the version tag is to be generated. Defaults to `'v%s'` which eventually align with tag\_pattern property of puppet-blacksmith, thus changelog is referring to the correct version tags and compare URLs. |
|github_site|Override built-in default for public GitHub. Useful for GitHub Enterprise and other. (Example: `github_site = https://git.domain.tld`) |
|github_endpoint|Override built-in default for public GitHub. Useful for GitHub Enterprise and other. (Example: `github_endpoint = https://git.domain.tld/api/v4`) |
|gitlab|Allows to work with GitLab (Pending ![GitLab support](https://github.com/github-changelog-generator/github-changelog-generator/pull/657)). (Example: `gitlab = true` # Boolean default is false)|
|default\_disabled\_lint\_checks| Defines any checks that are to be disabled by default when running lint checks. As default we disable the `--relative` lint check, which compares the module layout relative to the module root. _Does affect **.puppet-lint.rc**._ |
|extra\_disabled\_lint\_checks| Defines any checks that are to be disabled as extras when running lint checks. No defaults are defined for this configuration. _Does affect **.puppet-lint.rc**._ |
|extras|An array of extra lines to add into your Rakefile. As an alternative you can add a directory named `rakelib` to your module and files in that directory that end in `.rake` would be loaded by the Rakefile.|
|linter\_options| An array of options to be passed into linter config. _Does affect **.puppet-lint.rc**._ |
|linter\_fail\_on\_warnings| A boolean indicating if the linter should exit non-zero on warnings as well as failures. _Does affect **.puppet-lint.rc**._ |

### .rubocop.yml

>[RuboCop](https://github.com/bbatsov/rubocop) is a Ruby static code analyzer. We use Rubocop to enforce a level of quility and consistancy within Ruby code. Rubocop can be configured within .rubocop.yml which is located in the root directory of the code repository. Rubocop works by defining a sanitized list of cops that'll cleanup a code base without much effort, all of which support autocorrect and that are fairly uncontroversial across wide segments of the Community.

| Key            | Description   |
| :------------- |:--------------|
|include\_todos|Allows you to use rubocop's "TODOs" to temporarily skip checks by setting this to `true`. See rubocop's `--auto-gen-config` option for details. Defaults to `false`.|
|selected\_profile|Allows you to define which profile is used by default, which is set to `strict`, which is fully defined within the `profiles` section.|
|default\_configs |Allows you to define the default configuration of which cops will run. Includes the full name of the cop followed by a description of it and an enforced style. Can also make use of the key `excludes` to exclude any files from that specific cop.|
|cleanup\_cops    |Defines a set of cleanup cops to then be included within a rubocop profile. Cops are defined by their full name, and further configuration can be done by specifying secondary keys. By default we specify a large amount of cleanup cops using their default configuration.|
|profiles         |Defines the profiles that can be enabled and used within rubocop through the `selected_profile` option. By default we have set up three profiles: cleanups\_only, strict, hardcore and off.|

### Gemfile

>A Gemfile is a file we create which is used for describing gem dependencies for Ruby programs. All modules should have an associated Gemfile for installing the relevant gems. As development and testing is somewhat consistant between modules we have used the template to define a set of gems relevant to these processes.

| Key            | Description   |
| :------------- |:--------------|
|required|Allows you to specify gems that are required within the Gemfile. Gems can be defined here within groups, for example we use the :development gem group to add in several gems that are relevant to the development of any module and the :system_tests gem group for gems relevant only to acceptance testing.|
|optional|Allows you to specify additional gems that are required within the Gemfile. This key can be used to further configure the Gemfile through assignment of a value in the .sync.yml file.|

>Within each Gem group defined using the options above one or more gem item definitions may be listed in an array. Each item in that array must be a gem item hash.

| Gem Item Hash Keys | Description  |
| :----------------- |:-------------|
|gem|Required option specifying the gem name.|
|version|Required option to specify version or range of versions required using [RubyGem version syntax](https://guides.rubygems.org/patterns/#pessimistic-version-constraint).|
|platforms|Defines an array of platforms for which the Gem should be included. See the [Gemfile platform guide](https://bundler.io/man/gemfile.5.html#PLATFORMS) for a list of valid platforms.|
|git|If required, specify a specific Git repository in which this Gem is located. See the [Bundler docs](https://bundler.io/man/gemfile.5.html#GIT) for details.|
|branch|Optionally specify a branch to use if using the `git` option. Defaults to `master`.|
|ref|Optionally specify an arbitrary valid Git reference to use for the module version.|
|source|Specify an alternate Rubygems repository to load the gem from.|
|from_env|Specifies an environment variable containing either a Rubygem version specification indicating the version to use OR a URL indicating the location from which to load the gem.|
|condition|An optional string containing a Ruby-code conditional controlling if this gem will be processed in the Gemfile.|

### spec/default_facts.yml

> The spec/default_facts.yml file contains a list of facts to be used by default when running rspec tests

| Key            | Description   |
| :------------- |:--------------|
|concat_basedir|Overrides the concat_basedir fact's value in the base template. Defaults to "/tmp".|
|ipaddress|Overrides the ipaddress fact's value in the base template. Defaults to "172.16.254.254".|
|is_pe|Overrides the is_pe fact's value in the base template. Defaults to false.
|macaddress|Overrides the macaddress fact's value in the base template. Defaults to "AA:AA:AA:AA:AA:AA".
|extra_facts|List of extra facts to be added to the default_facts.yml file. They are in the form: "`name of fact`: `value of fact`"|

### spec/spec_helper.rb

> The spec/spec_helper.rb file contains setup for rspec tests

| Key            | Description   |
| :------------- |:--------------|
|default_facter_version|Sets the [`default_facter_version`](https://github.com/mcanevet/rspec-puppet-facts#specifying-a-default-facter-version) rspec-puppet-facts parameter.|
|hiera_config|Sets the [`hiera_config`](http://rspec-puppet.com/documentation/configuration/#hiera_config) rspec-puppet parameter.|
|hiera_config_ruby|Sets the [`hiera_config`](http://rspec-puppet.com/documentation/configuration/#hiera_config) rspec-puppet parameter. A ruby expression returning the path to your hiera.yaml file. `hiera_config` takes precedence if both values `hiera_config` and `hiera_config_ruby` are specified. |
|mock_with|Defaults to `':mocha'`. Recommended to be set to `':rspec'`, which uses RSpec's built-in mocking library, instead of a third-party one.|
|spec_overrides|An array of extra lines to add into your `spec_helper.rb`. Can be used as an alternative to `spec_helper_local`|
|strict_level| Defines the [Puppet Strict configuration parameter](https://puppet.com/docs/puppet/5.4/configuration.html#strict). Defaults to `:warning`. Other values are: `:error` and `:off`. `:error` provides strictest level checking and is encouraged.|
|strict_variables| Defines the [Puppet Strict Variables configuration parameter](https://puppet.com/docs/puppet/5.4/configuration.html#strict_variables). Defaults to `true` however due to `puppetlabs_spec_helper` forced override (https://github.com/puppetlabs/puppetlabs_spec_helper/blob/070ecb79a63cb8fa10f46532c413c055e2697682/lib/puppetlabs_spec_helper/module_spec_helper.rb#L71). Set to `false` to align with true default or with `STRICT_VARIABLES=no` environment setting.|
|coverage_report|Enable [rspec-puppet coverage reports](https://rspec-puppet.com/documentation/coverage/). Defaults to `false`|
|minimum_code_coverage_percentage|The desired code coverage percentage required for tests to pass. Defaults to `0`|

## Making local changes to the template

> While we provide a basic template it is likely that it will not match what you need exactly, as such we allow it to be altered or added to through the use of the `.sync.yml` file.

### Adding configuration values

Values can be added to the data passed to the templates by adding them to your local `.sync.yml` file, thus allowing you to make changes such as testing against additional operating systems or adding new rubocop rules.

To add a value to an array simply place it into the `.sync.yml` file as shown below, here I am adding an additional unit test run against Puppet 4:

```yaml
.travis.yml:
  includes:
    - env: PUPPET_GEM_VERSION="~> 4.0" CHECK=parallel_spec
      rvm: 2.1.9
```

### Removing default configuration values

Values can be removed from the data passed to the templates using the [knockout prefix](https://www.rubydoc.info/gems/puppet/DeepMerge) `---` in `.sync.yml`.

To remove a value from an array, prefix the value `---`. For example, to remove
`2.5.1` from the `ruby_versions` array in `.travis.yml`:

```yaml
.travis.yml:
  ruby_versions:
    - '---2.5.1'
```

To remove a key from a hash, set the value to `---`. For example, to remove the
`ipaddress` fact from `spec/default_facts.yml`:

```yaml
spec/default_facts.yml:
  ipaddress: '---'
```

## Setting custom gems in the Gemfile

To add a custom internal `puppet-lint` plugin served from an internal Rubygems source, add
an entry similar to the following in `.sync.yml` file and run `pdk update`.

```yaml
Gemfile:
  optional:
    ':development':
      - gem: 'puppet-lint-my_awesome_custom_module'
        version: '>= 2.0'
        source: 'https://myrubygems.example.com/'
```

## Enabling Beaker system tests

To enable the ability to run Beaker system tests on your module, add the
following entry to your `.sync.yml` and run `pdk update`.

```yaml
Gemfile:
  required:
    ':system_tests':
      - gem: 'puppet-module-posix-system-r#{minor_version}'
        platforms: ruby
      - gem: 'puppet-module-win-system-r#{minor_version}'
        platforms:
          - mswin
          - mingw
          - x64_mingw
.gitlab-ci.yml:
  bundler_args: --with system_tests --path vendor/bundle --jobs $(nproc)
  beaker: true
appveyor.yml:
  appveyor_bundle_install: "bundle install --jobs 4 --retry 2 --with system_tests"
```

## Further Notes <a name="notes"></a>

Please note that the early version of this template contained only a 'moduleroot' directory, and did not have a 'moduleroot\_init'. The PDK 'pdk new module' command will still work with templates that only have 'moduleroot', however the 'pdk convert' command will fail if the template does not have a 'moduleroot_init' directory present. To remedy this please use the up to date version of the template.

[legacy_facts_doc]: https://puppet.com/docs/facter/latest/core_facts.html#legacy-facts
[legacy_facts_pl_plugin]: https://github.com/mmckinst/puppet-lint-legacy_facts-check
