# PHP Template Inheritance

**Template Inheritance** is an extremely useful technique for making reusable HTML layouts for a site. It is much more flexible than alternative techniques, such as “including” common elements of a page (like a header and footer file). The concept has been around for a while, most notably in the [Django](http://www.djangoproject.com/) template engine. Unlike other libraries, *PHP Template Inheritance* lets you write everything in straight PHP. **There is no need to learn another template language**.

## Tutorial

1. [Basic Inheritance](#basic-inheritance) (*startblock, endblock*)
2. [More on Blocks](#more-on-blocks) (*emptyblock*)
3. [Default Block Content & Overriding](#defaults-overriding) 
4. [Every Block is Always Executed!](#always-executed)
5. [The Super Block](#super-block) (*superblock, getsuperblock*)
6. [Nested Blocks](#nested-blocks)
7. [Block Filters](#block-filters)
8. [Under the Hood](#under-the-hood)
9. [Flushing & Capturing Output](#flushing-capturing) (*flushblocks*)
10. [Base With No Blocks](#base-no-blocks) (*blockbase*)

## 1. Basic Inheritance <a name="basic-inheritance"></a>

Template Inheritance usually involves two separate templates, each in their own file: the *parent* template and the *child*template. The parent contains the HTML skeleton and markers for where content should go. These markers are called *blocks*. The child then “fills-in” the blocks with content. Example:

```php
// BASE.PHP (the parent)
<?php require_once 'ti.php' ?>
<html>
<body>
  <h1>
    <?php startblock('title') ?>
    <?php endblock() ?>
  </h1>
    
  <div id='article'>
    <?php startblock('article') ?>
    <?php endblock() ?>
  </div>
</body>
</html>
    
    
// PAGE.PHP (the child)    
<?php include 'base.php' ?>

<?php startblock('title') ?>
   This is the title
<?php endblock() ?>

<?php startblock('article') ?>
   This is the article
<?php endblock() ?>
    
    
// PAGE.PHP's output
<html>
<body>
  <h1>
     This is the title
  </h1>

  <div id='article'>
     This is the article
  </div>
</body>
</html>
```

The child is always responsible for declaring its parent. This is done with a simple `include` statement at the beginning of the child file.

When declaring a parent, it is also possible to use `require`. However, `include` is preferred as it does not throw a fatal error if the parent does not exist.

## 2. More on Blocks <a name="more-on-blocks"></a>

In the previous example, both the parent and child templates contain blocks. A block begins with a call to the `startblock` function, passing in the name of the block, and ends with a call to `endblock`. For added clarity, you can optionally pass the name to `endblock` as well.

You will often encounter blocks that contain no content (such as in base.php above). This is the norm for parent templates in general. A good shorthand for this is the `emptyblock` function.

The following three ways of defining an empty block are all equivalent:

```php
<?php startblock('article') ?>
<?php endblock() ?>
    
<?php startblock('article') ?>
<?php endblock('article') ?>
    
<?php emptyblock('article') ?>    
```

## 3. Default Block Content & Overriding <a name="defaults-overriding"></a>

You won’t always want your child template to fill in every one of your parent’s blocks. Sometimes you’ll want the parent to contribute content instead. This content is called *default* content. Example:

```php
// BASE.PHP
<?php require_once 'ti.php' ?>
<html>
<body>
  <div id='main'>
    <?php emptyblock('main') ?>
  </div>
  <div id='footer'>
    <?php startblock('footer') ?>
       Copyright 2010
    <?php endblock() ?>
  </div>
</body>
</html>
    
    
// PAGE.PHP
<?php include 'base.php' ?>

<?php startblock('main') ?>
   The main content
<?php endblock() ?>


// PAGE.PHP's output
<html>
<body>
  <div id='main'>
     The main content
  </div>
  <div id='footer'>
     Copyright 2010
  </div>
</body>
</html>    
```

As you can see, the child doesn’t define a footer block, so the parent’s footer gets used instead.

However, if the child *did* decide to define a footer, it would *override* the parent’s. Example:

```php
// BASE.PHP
<?php require_once 'ti.php' ?>
<html>
<body>
  <div id='main'>
    <?php emptyblock('main') ?>
  </div>
  <div id='footer'>
    <?php startblock('footer') ?>
       Copyright 2010
    <?php endblock() ?>
  </div>
</body>
</html>
    
    
// PAGE.PHP
<?php include 'base.php' ?>

<?php startblock('main') ?>
   The main content
<?php endblock() ?>

<?php startblock('footer') ?>
   Custom footer!!!
<?php endblock() ?>
    
    
// PAGE.PHP's output
<html>
<body>
  <div id='main'>
     The main content
  </div>
  <div id='footer'>
     Custom footer!!!
  </div>
</body>
</html>    
```

Default block content is useful if you are not sure an area of content will change in the parent template. When in doubt, wrap the content in `startblock` and `endblock` tags, and you can always override it later.

## 4. Every Block is Always Executed! <a name="always-executed"></a>

It is important to note that `startblock` and `endblock` are not control structures, so they cannot prevent code from executing. *All code between startblock and endblock will always be executed!…* regardless of whether the block is overridden or not.

If you have a block with high resource demands (such as database calls) and you only want to it to execute if its content is guaranteed to be in the output (in other words, not overridden), then you might need to restructure your code.

## 5. The Super Block <a name="super-block"></a>

We have seen that a parent’s block can either be kept as-is, or overridden by a child’s block. However, what if you wanted to keep a parent’s block *and* incorporate additional content? This is possible by calling the `superblock`function from within a block. It will insert the overriden block’s content into the output. Example:

```php
// BASE.PHP
<?php require_once 'ti.php' ?>
<html>
<body>
  <?php startblock('top') ?>
     <h1>The Main Title</h1>
  <?php endblock() ?>
</body>
</html>

    
// PAGE.PHP
<?php include 'base.php' ?>

<?php startblock('top') ?>
   <?php superblock() ?>
   <h2>The Second Title</h2>
<?php endblock() ?>
    
    
// PAGE.PHP's output
<html>
<body>
   <h1>The Main Title</h1>
   <h2>The Second Title</h2>
</body>
</html>    
```

An additional function exists called `getsuperblock`. This is the same as `superblock` except it *returns* the super block’s content as a string instead of sending it to the output.

## 6. Nested Blocks <a name="nested-blocks"></a>

A block’s content usually consists of HTML, but it can also consists of other blocks as well. These blocks-within-blocks are called *nested* blocks, and they function very similarly to normal blocks:

```php
// BASE.PHP
<?php require_once 'ti.php' ?>

<html>
<body>
  <div id='content'>
    <?php emptyblock('content') ?>
  </div>
</body>
</html>
    
    
// 2COL.PHP
<?php include 'base.php' ?>

<?php startblock('content') ?>
   <table>
     <tr>
       <td>
         <?php emptyblock('left') ?>
       </td>
       <td>
         <?php emptyblock('right') ?>
       </td>
     </tr>
   </table>
<?php endblock() ?>    
    
    
// PAGE.PHP
<?php include '2col.php' ?>

<?php startblock('left') ?>
   left content
<?php endblock() ?>

<?php startblock('right') ?>
   right content
<?php endblock() ?>
    
    
// PAGE.PHP's output
<html>
<body>
<div id='content'>
   <table>
     <tr>
       <td>
         left content
       </td>
       <td>
         right content
       </td>
     </tr>
   </table>
</div>
</body>
</html>    
```

Note, this example also demonstrates how there can be more than two parents involved in template inheritance. In this case there are *three*. **page.php** inherits from **2col.php**, which in turn inherits from **base.php**!

## 7. Block Filters <a name="block-filters"></a>

You’ve seen the basic usage of `startblock` and `endblock`, but you have not yet seen them used with a *filter function*. This is an optional second argument for `startblock`.

A filter function allows you to modify a block’s content before it is sent to the output. It is a function that accepts a single string argument (the content of the block) and must return a string that will be sent to the output. Here is a barebones example:

```php
// PAGE.PHP
<?php  
  require_once 'ti.php';

  function exclaim($s) {
     return trim($s) . "!!!";
  }
?>
    
<html>
<body>
  <div id='content'>
  <?php startblock('content', 'exclaim') ?>
     This is the content
  <?php endblock() ?>
  </div>
</body>
    
    
// PAGE.PHP's output
<html>
<body>
  <div id='content'>
    This is the content!!!
  </div>
</body>
</html>    
```

In page.php’s call to `startblock`, the string `'exclaim'` is being passed, signifying that the filter function is a global function of the same name. In PHP version 5.3 or greater, you may pass an [anonymous function](http://php.net/manual/en/functions.anonymous.php).

Contrary to this example, it is usually good to keep your filter functions in a separate PHP file and then `include_once`them from your templates.

Filter functions are useful if you want your content to be in a different markup format. For example, you could include the [PHP markdown library](http://michelf.com/projects/php-markdown/), and have your filter function be `'markdown'`.

Multiple filters are possible by passing a comma-delimited string like `'markdown,exclaim'` or an array of strings/functions like `array('markdown', 'exclaim')`.

## 8. Under the Hood <a name="under-the-hood"></a>

So how does *PHP Template Inheritance* do its job behind the scenes? At the first sign of a block, it initializes an [output buffer](http://www.php.net/manual/en/book.outcontrol.php). It then continues normal PHP execution, keeping tabs on every block it encounters. When the page request is done, it compiles all the blocks together and sends them to standard out.

It also does some tricky stuff with the [debug_backtrace](http://php.net/manual/en/function.debug-backtrace.php) function to analyze the call stack. To see how everything works, you can browse the [source code on GitHub](http://github.com/asherber/phpti/blob/master/src/ti.php).

*PHP Template Inheritance* has very good performance because it utilizes PHP’s native output buffers and does not rely on any kind of expensive parsing.

## 9. Flushing and Capturing Output <a name="flushing-capturing"></a>

Because the library utilizes an output buffer, you might be wondering how you can “break out” of the output buffer and return to PHP’s normal rules of output (where output is directly sent to the browser). This can be done by calling `flushblocks`, a function with zero arguments.

As with the PHP language, all output is sent to the browser by default. However, what if you wanted to capture all output into a *string*? This is entirely possible by utilizing the previously described `flushblocks` function along with PHP’s [output buffer functions](http://www.php.net/manual/en/book.outcontrol.php). Example:

```php
<?php
  require_once 'ti.php';

  ob_start();
  include 'yourtemplate.php';
  flushblocks();
  $s = ob_get_clean();

  // $s now holds the output!
?>
```

## 10. Base With No Blocks <a name="base-no-blocks"></a>

*PHP Template Inheritance* strives to provide a very intuitive API. However, there is one situation in which it might get confused, so you’ll need to give it a hint. This situation occurs when you have a base template that has no blocks. **You must call the blockbase() function** in the base template in order to make things work. Example:

```php
// BASE.PHP
<?php require_once 'ti.php' ?>

<?php blockbase() ?>
    
<html>
<body>
  There are no blocks!
</body>
</html>
    
    
// PAGE.PHP
<?php include 'base.php' ?>

<?php startblock('content') ?>
   Block that doesn''t do anything
<?php endblock() ?>


// PAGE.PHP's output
<html>
<body>
  There are no blocks!
</body>
</html>       
```
