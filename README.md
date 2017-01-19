## Welcome
# Using Lua in Ruby
## A simple class/helper to evaluate Lua code
The rufus-lua gem is interesting and useful. This is a "manager" class.
```
class LuaEngine
  attr_accessor :_lua, :errors
  def self.instance
    (@_instance == nil) ? @_instance = LuaEngine.new : @_instance
  end
  def initialize
    self.errors = []
    self._lua = Rufus::Lua::State.new
    expose_api
    load_submitted_lua_code
    unregister_functions
    puts "Lua engine initialized..."
  end

  def load_submitted_lua_code
    math_round = "function math.round(x, n) n = math.pow(10, n or 0) x = x * n  if x >= 0 then x = math.floor(x + 0.5) else x = math.ceil(x - 0.5) end return x / n end"
    self._lua.eval(math_round)
    #strings = load_from_database TODO: Load Lua code from aother source
    #strings.each do |s|
    #  self._lua.eval(s.code)
    #end
  end

  def expose_api
    self._lua.function 'mynamespace.awesome_function' do |table|
      RubyClass.awesome_function(table.values) #This is just an example
    end
  end
  def unregister_functions
    [:io, :print, :loadfile, :os, :dofile, :collectgarbage, :_G, :getfenv, :getmetatable, :setfenv, :setmetatable, 'string.rep'].each do |badfunc|
      self._lua.eval("#{badfunc} = nil")
    end
    self._lua.eval("function string.rep(count) return 'rep not supported'; end")
  end
end
```
					        


