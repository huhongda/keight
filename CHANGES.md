Changes
=======

Release 0.3.0 (2016-10-13)
--------------------------

### Enhancements

* URL path parameter format can accept parameter type (`int`, `str`, `date`).  
  ex: `/api/books/{id:int}`, `/blog/{entry_date:date}`

* `K8::Util::TemporaryFile` class added. It allows you to create temporary
  file which will be removed automatically *after* sending response body.  
  See document of `K8::Util::TemporaryFile` for details.

* `K8::Util::ShellCommand` class added. It allows you to send command output
  as response body. For example, execute SQL, convert into CSV, and send it
  as response.  
  See document of `K8::Util::ShellCommand` for details.

* `@action_args` is available in `K8::Action` class.  
  `@action_args` represents URL path parameter arguments.  
  For example, `@action_args == [123]` when urlpath pattern is `/books/{id}`
  and request path is `/books/123`.

* `@req.params_form()`, `@req.params_multipart()` and `@req.params_json()`
  now take max content length.  
  ex:

      strs = @req.params_form(10*1024*104)  # max: 10MB
      strs = @req.params_form               # same as above

      strs, files = @req.params_multipart(100*1024*1024)  # max: 100MB
      strs, files = @req.params_multipart                 # same as above

      hash = @req.params_json(10*1024*104)  # max: 10MB
      hash = @req.params_json               # same as above

* Add `K8::ActionMapping#urlpath_rexp`.
  You can confirm regexp of action mapping by that attribute.  
  ex:

      ## what regexp is used for request routing?
      mappings = [
        ['/api', [
          ['/books',    BooksAction],
          ['/authors',  AuthorsAction],
        ]
      ]
      p K8::ActionMapping.new(mapping).urlpath_rexp

* Regard '.*' at end of urlpath pattern as extension pattern.  
  ex:

      class BooksAPI < K8::Action

        ## ex: '.*' is same as '{_:<(?:\.\w+)?>}'
        mapping '.*'     , :GET=>:do_index

        ## ex: '/{id}.*' is same as '/{id}{_:<(?:\.\w+)?>}'
        mapping '/{id}.*', :GET=>:do_show

        def do_index
          p @req.path_ext    #=> ex: '.json', '.html', etc
        end
        def do_show(id)
          p @req.path_ext    #=> ex: '.json', '.html', etc
        end
      end

* Define `@req.path_ext` which returns such as `.json` or `.html`.

* Define `@resp.status_line` which returns such as `"200 OK"` or `"302 Found"`.

* `@resp.set_cookie()` now accepts Time object as `expires` keyword arg.

* New HTTP response status codes are added.  
  #

      103 Checkpoint                           # Unofficial
      306 Switch Proxy                         # -
      308 Permanent Redirect                   # RFC 7538
      421 Misdirected Request                  # RFC 7540
      428 Precondition Required                # RFC 6585
      429 Too Many Requests                    # RFC 6585
      431 Request Header Fields Too Large      # RFC 6585
      444 No Response                          # nginx
      451 Unavailable For Legal Reasons        # -
      495 SSL Certificate Error                # nginx
      496 SSL Certificate Required             # nginx
      497 HTTP Request Sent to HTTPS Port      # nginx
      499 Client Closed Request                # nginx
      509 Bandwidth Limit Exceeded             # Apache Web Server/cPanel
      511 Network Authentication Required      # RFC 6585
      520 Unknown Error                        # CloudFlare
      521 Web Server Is Down                   # CloudFlare
      522 Connection Timed Out                 # CloudFlare
      523 Origin Is Unreachable                # CloudFlare
      524 A Timeout Occurred"                  # CloudFlare
      525 SSL Handshake Failed                 # CloudFlare
      526 Invalid SSL Certificate              # CloudFlare

* `k8rb cdnjs` now calls CDNJS.com api instead of scraping HTML page.


### Incomppatible changes

* URL path parameter format is changed.  
  (old) `/books/{id:\d+}`  
  (new) `/books/{id:int}` or `/books/{id:int<\d+>}`  
  See document for details.

* Rename `@req.method` to `@req.meth`.  
  `@req.method` is also available, but not recommended. Use `@req.meth`.

* Rename `K8::BaseAction[]#method` to `K8::BaseAction[]#meth`.  
  ex:

      p BooksAction[:do_show].meth     #=> :GET
      p BooksAction[:do_update].meth   #=> :PUT
      p BooksAction[:do_delete].meth   #=> :DELETE

* Rename `K8::BaseAction[]#urlpath()` to `BooksAction[]#path()`.  
  ex:

      p BooksAction[:do_show].path(123)     #=> "/api/books/123"
      p BooksAction[:do_update].path(123)   #=> "/api/books/123"
      p BooksAction[:do_delete].path(123)   #=> "/api/books/123"

* Rename `@current_action` to `@action_name` in `K8::Action` class.
  It represents current action name, such as `:do_index` or `:do_show`.

* `@req.params` now raises `K8::PayloadParseError` for JSON data,
  because `@req.params` should return hash object of string, but
  JSON data contains non-string values.
  Use `@req.json` instead fo `@req.params`.

* `@req.params` now raises `K8::PayloadParseError` for multipart data,
  because `@req.params` should return hash object of string, but
  multipart form contains file data which is not a string value.
  Use `@req.multipart` instead fo `@req.params`.

* Rename `K8::Request` class to `K8::RackRequest`.

* Rename `K8::Response` class to `K8::RackResponse`.

* Rename `K8::REQUEST_CLASS` to `K8::RackApplication::REQUEST_CLASS`.
* Rename `K8::RESPONSE_CLASS` to `K8::RackApplication::RESPONSE_CLASS`.
* Move `K8::ActionMapping#lookup()` to `K8::RackApplication` class.
* Remove 'K8::DefaultPatterns' class.
* Remove 'K8::DEFAULT_PATTERNS' object.

* `k8rb mapping` command is removed. Use `rake mapping` instead.
* `k8rb configs` command is removed. Use `rake configs` instead.

* Remove `K8::BaseConfig` and `K8::SecretValue` classes.
  Use 'benry-config' gem instead.

* Remove 'keight/skeleton/*' files. File are moved to
  https://github.com/kwatch/keight-ruby-boilerpl8/releases .

* Remove `K8::RackApplicaiton#lookup_autoredirect_location()`.


### Changes

* Rewrite `K8::ActionMapping` class entirely.
* Remove `K8::ActionMethodMapping` class.
* Rewrite `K8::RackRequest#params_form`, `#params_multipart` and `#params_json`.
* Move session initialization code from `K8::BaseAction` to `K8::Action`.
* Remove dependency to `baby_erubis` gem.


Release 0.2.0 (2016-01-06)
--------------------------

* [change] `K8::RackApplication#mount()` is removed.  
  Use `K8::RackApplication.new([...])` instead.

* [change] `k8rb init` is renamed to `k8rb project`.  
  ex:

      $ k8rb project myapp1
      $ cd myapp1

* [enhance] Auto redirection support.
  For example, `GET /books` is defined and `GET /books/` is requested,
  then it will be redirected to `GET /books`.

* [enhance] Performance improved for variable urlpath.

* [enhance] Implement urlpath helpers.  
  ex:

      p BookAPI[:do_update].method          #=> :PUT
      p BookAPI[:do_update].urlpath(123)    #=> '/api/books/123'
      p BookAPI[:do_update].form_action_attr(123)  #=> '/api/books/123?_method=PUT'

* [change] `k8rb mapping` now prints output in text format, not YAML format.

* [enhance] `k8rb mapping` supports `--format=FORMAT` option.  
  ex:

      $ k8rb mapping --format=text   # or yaml/json/javascript/jquery/angular

* [enhance] `k8rb` command supports `cdnjs` action which download JavaScript
  libraries from cdnjs.com.  
  ex:

      $ k8rb cdnjs                   # list library
      $ k8rb cdnjs 'jquery*'         # search library
      $ k8rb cdnjs jquery            # list versions
      $ k8rb cdnjs jquery 2.1.4      # download library

* [change] `k8rb init` command downloads jquery and modernizr from cdnjs.com.

* [change] `K8::Mock` and `K8::TestApp` are removed.
  Use rack-test_app gem instead.

* [bugfix] Document fixed.

* [internal] Remove `K8::ActionClassMapping`, `K8::ActionRouter` and
  `K8::ActionFinder` classes.

* [internal] Define new class `K8::ActionMapping` instead of remove classes.


Release 0.1.0 (2015-10-27)
--------------------------

* Public release


Release 0.0.1 (2015-10-26)
--------------------------

* Test release
