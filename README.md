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

## Extending DelayedJob::ActiveRecord::Backend
[Using collectiveidea/delayed_job_active_record](https://github.com/collectiveidea/delayed_job_active_record)
A simple way to make additional checks when retrieving delayed_job records from database
*Your extensions.rb file
``` ruby
module Delayed
  module Backend
    module ActiveRecord
      class Job
         
        # Overriding
        # This method extracts the next job in the schedule
        def self.reserve_with_scope_using_optimized_sql(ready_scope, worker, now)
          quoted_table_name = connection.quote_table_name(table_name)
          subquery_sql = ready_scope.limit(1).lock(true).select("id").to_sql
          reserved = find_by_sql(["UPDATE #{quoted_table_name} SET locked_at = ?, locked_by = ? WHERE id IN (#{subquery_sql}) RETURNING *", now, worker.name])
          candidate = reserved[0]
          #my_custom_function_to_manipulate(candidate)
        end
      end
    end
  end
end
```

## Obtain the mode of a Lua table
``` lua
-- Returns a table of values.
-- Works on anything (not just numbers).
function stats.mode( t )
    local counts={}
    for k, v in pairs( t ) do
        if counts[v] == nil then
            counts[v] = 1
        else
            counts[v] = counts[v] + 1
        end
    end
    local biggestCount = 0
    for k, v  in pairs( counts ) do
        if v > biggestCount then
            biggestCount = v
        end
    end
    local temp={}
    for k,v in pairs( counts ) do
        if v == biggestCount then
            table.insert( temp, k )
        end
    end
    return temp
end
```
