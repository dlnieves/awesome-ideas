# Welcome
## Using Lua in Ruby

### A simple class/helper to evaluate Lua code

The [rufus-lua](https://github.com/jmettraux/rufus-lua) gem is interesting and useful. You can build an expression or rules engine using the Lua programming language

```ruby
class LuaEngine
  def self.instance
    (@_instance == nil) ? @_instance = LuaEngine.new : @_instance
  end
  def initialize
    @_lua = Rufus::Lua::State.new #This is the Lua State
    expose_api
    load_submitted_lua_code
    unregister_functions
    puts "Lua engine initialized..."
  end

  #Preload some lua functions and make them available
  def load_submitted_lua_code
    math_round = "function math.round(x, n) n = math.pow(10, n or 0) x = x * n  if x >= 0 then x = math.floor(x + 0.5) else x = math.ceil(x - 0.5) end return x / n end"
    @_lua.eval(math_round)
    #strings = load_from_database TODO: Load Lua code from another source
    #strings.each do |s|
    #  self._lua.eval(s.code)
    #end
  end

  #You can expose your api
  def expose_api
    self._lua.function 'mynamespace.awesome_function' do |table|
      RubyClass.awesome_function(table.values) #This is just an example
    end
  end
  
  #You can add here some security considerations
  def unregister_functions
    [:io, :print, :loadfile, :os, :dofile, :collectgarbage, :_G, :getfenv, :getmetatable, :setfenv, :setmetatable, 'string.rep'].each do |badfunc|
      @_lua.eval("#{badfunc} = nil")
    end
    @_lua.eval("function string.rep(count) return 'rep not supported'; end")
  end
end
```
	
