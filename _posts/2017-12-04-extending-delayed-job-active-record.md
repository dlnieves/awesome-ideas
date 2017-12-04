# Extending DelayedJob::ActiveRecord::Backend
[Using collectiveidea/delayed_job_active_record](https://github.com/collectiveidea/delayed_job_active_record)
A simple way to make additional checks when retrieving delayed_job records from database
Your extensions.rb file
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
          #my custom code to manipulate the current Job
        end
      end
    end
  end
end
```
