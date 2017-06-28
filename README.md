# chef-alfresco-best-practices

## Undestanding chef

Valuable reads:

- [Chef's Two Pass Model](https://coderanger.net/two-pass/)
- [Attribute Precedence](https://docs.chef.io/attributes.html#attribute-precedence)

## Repo Structure

- Create a new git repository for each cookbook. This is known as The [Berkshelf Way](https://www.slideshare.net/opscode/the-berkshelf-way-20882903).

## Attributes

- **Avoid node.set**

Use node.default (or maybe node.override) instead of node.set because node.set is an alias for node.normal. Normal data is persisted on the node object. Therefore, using node.set will persist data in the node object. If the code that uses node.set is later removed, if that data has already been set on the node, it will remain.

Default and override attributes are cleared at the start of the chef-client run, and are then rebuilt as part of the run based on the code in the cookbooks and recipes at that time.

node.set (and node.normal) should only be used to do something like generate a password for a database on the first chef-client run, after which it’s remembered (instead of persisted). Even this case should be avoided, as using a data bag is the recommended way to store this type of data.

- **Avoid derived attributes in attribute files.**

```
# Attributes:: default
// DO NOT DO THIS
default['my_cookbook']['version'] = '1.4.8'
default['my_cookbook']['url'] = "http://mysite/#{node['my_cookbook']['version']}.tar.gz"
```

If somebody overrides the version attribute, the url attribute won't be recomputed. It doesn't matter whether the attribute is overridden in an environment file or by another cookbook.

When you need to derive attributes, do it inside a recipe or provider. Or use the Delayed Interpolation:

```
default['my_cookbook']['version'] = '1.4.8'
default['my_cookbook']['url'] = lazy { "http://mysite/#{node['my_cookbook']['version']}.tar.gz" }
```

Also refer to [Poise Derived](https://github.com/poise/poise-derived), a Chef cookbook for defining lazily evaluated node attributes

## Cookbooks

- Use [Semantic Versioning](http://semver.org/) for your cookbook:

```
MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards-compatible manner, and
PATCH version when you make backwards-compatible bug fixes.
```

- Never ever decrement the version of a cookbook

- Include a README.md
- Include a TESTING.md
- Include a CONTRIBUTING.md
- Include a CHANGELOG.md

- if a functionality is common to more than a cookbook then move it to a shared cookbook [chef-alfresco-utils](https://github.com/Alfresco/chef-alfresco-utils)

## Recipes

- Private recipes should start with an underscore. The absence of an underscore is an indicator that the recipe is meant for use in run lists, or included from recipes in other cookbooks.
- If too long consider creating a new recipe
- Private recipes should start with an underscore. The absence of an underscore is an indicator that the recipe is meant for use in run lists, or included from recipes in other cookbooks.

## Templates

- Avoid using the node object within templates.

If you need to use attributes in a template, add them to the template resource using the variables parameter.

```
// GOOD:
Alice is <%= @age %> years old.
// BAD:
Alice is <%= node['my_cookbook']['alice']['age'] %> years old.
```

- Never refer to attributes from other cookbooks in your template.

## Conditionals

Understand Chef's Two Pass Model. Be aware of the difference between if statements and guards. One of them is handled during the compile phase, and the other during the execute phase.

Wrong:

```
random_name = ('a'..'z').to_a.shuffle[0,8].join
fileName = "/#{random_name}.txt"

file fileName do
  content 'bar'
end

if File.exist?(fileName)
  log 'File exists!'
else
  log 'File does not exist!'
end
```

Correct:

```
random_name = ('a'..'z').to_a.shuffle[0,8].join
fileName = "/#{random_name}.txt"

file fileName do
  content 'bar'
end

log 'File exists!' do
  only_if { File.exist?(fileName) }
end
```

## Resources

- Use **retry** when using resources downloading from the internet
- **create** vs **create_if_missing** action
- Always specify the file mode with a quoted 3-5 character string or 5 character numeric that defines the octal **mode**
  - Correct: ‘755’, ‘0755’, 00755
  - Wrong: 755
- When possible avoid **execute**, **bash**, **shell** resources
	- Always guarantee **idempotency**
- Specify resource **action**

## Useful Recipe DSL

- **shell_out**
- **edit_resource**
- **delete_resource**
- **find_resource**


## Tests
- Linting
  - **Foodcritic**
  - **Cookstyle**

- Unit Tests
  - **Chefspec** + **coveralls**

- Integration Tests
  - **Kitchen** + **Inspec**

## Sharing
[Chef Supermarket](https://supermarket.chef.io/users/alfresco)
