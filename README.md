# mineunit
Minetest core / engine libraries for regression tests

![mineunit](https://mineunit-badges.000webhostapp.com/mt-mods/mineunit/coverage)

Probably will not currently work with Windows so unless you want to help fixing things use Linux or similar OS.

### How to use mineunit
```bash
$ cd ~/.minetest/mods/my_minetest_mod
$ mkdir spec
$ git submodule add git@github.com:mt-mods/mineunit.git spec/mineunit
```

See examples below.

### Define world for tests

World can be replaced by calling `world.layout` with table containing nodes, this will reset previously created world layout.
You can also add more nodes without resetting previously added world layout by calling `world.add_layout` instead of `world.layout`.
```lua
world.layout({
	{{x=0,y=0,z=0}, "default:cobble"},
	{{x=0,y=1,z=0}, "default:cobble"},
	{{x=0,y=2,z=0}, "default:cobble"},
	{{x=0,y=3,z=0}, "default:cobble"},
})
```
Individual nodes can be added and removed with `world.set_node`:
```lua
world.set_node({x=0,y=0,z=0}, {name="default:stone", param2=0})
```
to remove node from world just set node to `nil`:
```lua
world.set_node({x=0,y=0,z=0}, nil)
```
to remove everything from world:
```lua
world.clear()
```

### Metadata

https://github.com/mt-mods/mineunit/issues/2
To set node metadata, simply call `minetest.get_meta(pos):set_string("hello", "world")` just like you would do in your mod.

### ItemStack

https://github.com/mt-mods/mineunit/issues/1
To create ItemStack, simply call `ItemStack("default:cobble 99")` just like you would do in your mod.

### Example mymod/spec/mymod_spec.lua file

Following comes with a lot of useless stuff just to show how to use some mineunit functionality

```lua
-- Load and configure mineunit
dofile("spec/mineunit/init.lua")

-- Load some mineunit modules
mineunit("core")
mineunit("player")
mineunit("protection")
mineunit("default/functions")

-- Load some fixtures for tests
fixture("nodes")

-- Load some mymod source files, you wanted to test these right?
sourcefile("init")

-- Maybe we need actual world for test?
world.layout({
	{{x=0,y=1,z=0}, "mymod:special_dirt"},
	{{x=0,y=0,z=0}, "mymod:special_dirt"},
	{{x=1,y=0,z=0}, "mymod:special_dirt"},
	{{x=0,y=0,z=1}, "mymod:special_dirt"},
	{{x=1,y=0,z=1}, "mymod:special_dirt"},
})

-- Protect some nodes
mineunit:protect({x=0,y=1,z=0}, "Sam")

-- Create few players
local player1 = Player("Sam", {interact=1})
local player2 = Player("SX", {interact=1})

-- Define tests for busted
describe("My test world", function()

	it("contains special_dirt", function()
		local node = minetest.get_node({x=0,y=0,z=0})
		assert.not_nil(node)
		assert.equals("mymod:special_dirt", node.name)
	end)

	it("allows Sam to dig dirt at y 1", function()
		assert.equals(false, minetest.is_protected({x=0,y=1,z=0}, player1:get_player_name()))
	end)

	it("protects dirt at y 1 from SX", function()
		assert.equals(true, minetest.is_protected({x=0,y=1,z=0}, player2:get_player_name()))
	end)

end)
```

### Known projects using mineunit

See following projects for more examples on how to use mineunit and what you can do with it

#### Technic Plus: simple, clean and straightforward tests.
* Network tests https://github.com/mt-mods/technic/tree/master/technic/spec
* GitHub workflow https://github.com/mt-mods/technic/blob/master/.github/workflows/busted.yml

#### Metatool: complex test setup. Mineunit development began here.
* Mineunit and global fixtures https://github.com/S-S-X/metatool/tree/master/spec
* Metatool API tests https://github.com/S-S-X/metatool/tree/master/metatool/spec
* Container tool behavior tests https://github.com/S-S-X/metatool/tree/master/containertool/spec
* GitHub workflow https://github.com/S-S-X/metatool/blob/master/.github/workflows/busted.yml
