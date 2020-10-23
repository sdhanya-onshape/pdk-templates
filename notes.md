### FAQ

I am using PDK docker container for all the below examples. You can choose to install PDK locally as well, the commands and the flags to be passed will remain the same.
- How to create a new module?

```
docker run --rm -it -v `pwd -P`:/code --workdir /code puppet/pdk new module mymodule --template-url=https://github.com/sdhanya-onshape/pdk-templates --template-ref=sdhanya-modifications
```

- What happens when I run the above command?

The above command will create a scaffolding for a new module which will work with PDK out of the box. On running the above command, you'll need to answer 4 questions which basically sets up the `metadata.json` for you. You can also choose to use the defaults and later edit the metadata file yourself. You can also add `--skip-interview` flag to the above command to skip it.

- Can I know what the questions are?

See the below example output that I got after I ran the above command -
```
pdk (INFO): Creating new module: mymodule
We need to create the metadata.json file for this module, so we're going to ask you 4 questions.
If the question is not applicable to this module, accept the default option shown after each question. You can modify any answers at any time by manually updating the metadata.json file.

[Q 1/4] If you have a Puppet Forge username, add it here.
We can use this to upload your module to the Forge when it's complete.
--> onshape

[Q 2/4] Who wrote this module?
This is used to credit the module's author.
--> sdhanya

[Q 3/4] What license does this module code fall under?
This should be an identifier from https://spdx.org/licenses/. Common values are "Apache-2.0", "MIT", or "proprietary".
--> Apache-2.0

[Q 4/4] What operating systems does this module support?
Use the up and down keys to move between the choices, space to select and enter to continue.
--> Debian based Linux

Metadata will be generated based on this information, continue? Yes
pdk (INFO): Using the specified template-url and template-ref 'https://github.com/sdhanya-onshape/pdk-templates#sdhanya-modifications'.
pdk (INFO): Module 'mymodule' generated at path '/code/mymodule'.
pdk (INFO): In your module directory, add classes with the 'pdk new class' command.
```

- Module structure that is setup on running the command,

```
$ tree
.
├── Gemfile
├── Gemfile.lock
├── README.md
├── Rakefile
├── examples
├── files
├── manifests
├── metadata.json
├── spec
│   ├── default_facts.yml
│   ├── fixtures
│   │   └── hiera
│   │       ├── common.yaml
│   │       └── hiera.yaml
│   └── spec_helper.rb
├── tasks
└── templates
8 directories, 9 files
```
The `Gemfile.lock` file, `examples`, & `tasks` directory are in `.gitignore` and will never get committed. We do not use these - so we can generally ignore them. I did not find a way to _not_ generate them. 

- How different is this fork from the default PDK template?

See the [diff](https://github.com/sdhanya-onshape/pdk-templates/pull/1/files)

- Is this template to be used for all the PDK commands?

No. This template is only needed for creating a new module or for converting an existing module to work with the PDK.

- Will validation and unit tests run out of the box?

Yes, although this template creates an empty skeleton, you can still run validation and test commands, which will pass.

For validation, (checks syntax and style for puppet, ruby, yaml, and metadata code) run the below command from the module.
```
docker run --rm -it -v `pwd -P`:/code --workdir /code puppet/pdk validate --parallel
```

For unit testing, (runs spec tests) run the below command from the module.
```
docker run --rm -it -v `pwd -P`:/code --workdir /code puppet/pdk test unit
```
