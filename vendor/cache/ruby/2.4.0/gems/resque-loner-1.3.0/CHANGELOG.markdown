1.3.0
--------------------------------
Enhancements and bug fixes from @mateusdelbianco.  Not allowing a job to
be executed immediately after execution, another from @mateusdelbianco!
Merged @andrejj's fix which removes an unused `#first` from a redis
multi call.  @kforsman created an internal helpers module to remove the
dependency on the deprecated `Resque::Helpers` module.  Added RuboCop,
SimpleCov, and CodeClimate, plus tweaked some stuff in Travis
configuration.


1.2.1
--------------------------------
Merged @aerodynamik's pull request. Enqueuing and marking as
enqueued as an atomic operation. 

1.2.0
--------------------------------
Thanks @unclebilly for your pull request. Resque-loner now supports
a maximum time for which a job should be unique. Just define @loner_ttl
in your job (or leave it at -1 to never expire) and after @loner_ttl
seconds your job can be enqueued again, even if an older one is still
marked as running.

1.1.0
--------------------------------
Merged in @ryansch's pull requests to clean up things a bit.
This removed the `enqueue_to` and `dequeue_from` methods from
resque-loner because it caused troubles with resque's own 
`enqueue_to`.

1.0.1
--------------------------------
Pulled in #8 and #9 so that removing empty queues
does not fail

1.0
---------------------------------
