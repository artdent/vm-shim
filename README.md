vm-shim
=======

Wan attempt to reproduce/polyfill/infill the node.js 
<code>vm#runIn<Some?>Context()</code> methods in browser ~ no guarantees.

+<code>vm#runInContext(code, context)</code> -- "done"
+<code>vm#runInNewContext(code)</code> -- todo
+<code>vm#runInThisContext(code)</code> -- todo

justify
-------

Goal here is to get something like <code>vm.createScript()</code> and/or 
<code>script||vm.runInContext()</code> method to run in the browser, in part to 
show that it really can be done, partly as a potential shim for browserify which 
tries to port node.js to the browser (with some caveats) ~ maybe.

browser tests
-------------

Using @pivotallabs' <a href='https://github.com/pivotal/jasmine'>jasmine</a> for the 
browser suite.

The *jasmine* test page is viewable on 
<a href='//rawgithub.com/dfkaye/vm-shim/master/test/browser/SpecRunner.html' 
   target='_new' title='opens in new tab or window'>
  rawgithub</a>.

node tests
----------

Out of curiosity, I've included a test suite to detect possible issues on Node, 
using @substack's <a href='https://github.com/substack/tape'>tape</a> module. 

Run these with:

    cd ./vm-shim
    npm test
  
or 
  
    node ./test/node-test.js
    
implementation
--------------

First-cut implementation is <code>vm.runInContext(code, context)</code>. The 
Function() constructor is at the core. The *code* param may be either a string 
or a function. The *context* param is a simple object with key-value mappings. 
For any key on the context, a new *var* for that key is prefixed to the code. The
code is passed in to Function() so that the keynames can't leak outside the new 
function's scope.

Currently the unit tests rely on the context param to contain a reference to the 
*t* or *test* object (when using *tape*), or the *expect* object (when running 
*jasmine*).

Example using *tape*:

    test("context attrs override external scope vars", function(t) {

      t.plan(1);

      var attr = "shouldn't see this";
       
      vm.runInContext(function(){
      
        test.equal(attr, 'ok');

      }, { attr: 'ok', test: t });
    });

Example using *jasmine*:

    it("context attrs override external scope vars", function() {

      var attr = "shouldn't see this";

      vm.runInContext(function(){

        expect(attr).toBe('ok');

      }, { attr: 'ok', expect: expect });
    });

 
__footgun__

Because JavaScript is a footgun where Function() is involved, debugger support 
will be necessary at some point, (as with using eval()).  I've added simple 
throw-error tests to find which engines return which helpful messages.


first success
-------------
Just noting for the record:

+ This idea emerged late at night 17 SEPT 2013 
+ First implemented with rawgithub approach 18 SEPT, 
+ Full success including objects as properties of the context argument 19 SEPT.
+ Breaking the usual TDD procedure:
  + Started with console statements and prayer ~ removed both for real unit tests
  + Tape tests added 20 SEPT.
  + Jasmine tests/page added 20 SEPT.
+ Error, and leakage tests added 21 SEPT.
