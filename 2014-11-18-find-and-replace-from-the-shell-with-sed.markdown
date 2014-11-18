This came up again while I upgrading our app from Bootstrap 2.x to 3.x.
Many of the class changes related to the grid system  are of the form ```span6``` ->
```col-md-6```, where a prefix before a number changes, but the number
stays the same.

Obviously, in the case of ```span```, you need to include the number
character in the regex you're matching against, so that you don't inadvertently change a
bunch of ```<span>``` tags into ```<col-md->``` tags.  At the same time, you need include the number
portion in what you replace the class with.  (I.e. ```class=span1```
becomes ```class=col-md-1```, ```class=span12``` becomes
```class=col-md-12```.)

The solution is to use regex captures to store the number portion and
reuse it in the replace portion of the substitution command.  There were
some problems getting this working with the [ack_sub]() tool I created
earlier (namely, both perl regex captures and bash arguments populate
variables named with dollar signs like ```$1```.)

After a little research, a popular unix utility called ```sed```
surfaced.  In this specific case, the command to change the classes is
as follows.

```
find . -type f -exec sed -i '' -E 's/span([0-9]+)/col-md-\1/g';
```

Let's break this down.  To start, sed accepts vi-style regex substitution
commands as its first argument.

```
echo "hello" | sed 's/ello/i/'
hi
```

It also accepts file name as a second argument.

```
echo "hello" > test.txt
sed 's/ello/i/' test.txt
hi
```

In this form, sed reads from the file, but does not edit it.

```
cat test.txt
hello
```

In order to actually edit the file, we'll pass sed the ```-i``` flag (for editing the file in-place).
With the ```-i``` flag, sed takes two arguments -- the extension to
append to backup a copy of the original file, followed by the command
string. If you want to edit the file without creating a backup, you can
pass an empty string as the first argument.  (WARNING: This is risky.  Only try this at
home if the files you're editing are backed up and/or under version
control.)

```
sed -i '' 's/ello/i/' test.txt

cat test.txt
hi
```
Great, now in order to use a more complex regular expression (with those
captures we discussed), we'll pass sed the ```-E``` flag.  In
combination with the ```-i``` flag, this goes in between the first and
second arguments.  With that in place, we can write out more complex
capture-using regex.  (NOTE: Sed regexes don't support the digit
matching excape character ```\d```.  We'll use the character range ```[0-9]``` instead.  ```\1``` includes the first capture in the replacement.)

```
echo "span6" > test.txt
echo "span12" >> test.txt
cat test.txt
span6
span12

sed -i '' -E 's/span([0-9])/col-md-\1/g' test.txt
cat test.txt
col-md-6
col-md-12
```

Beautiful.  Now how do we run this command on all the files in a
directory (in our case, everything in the views folder of our
application)?

Easy.  (Easily?)  We'll just use ```find``` with the ```-type f``` flag
for regular files.  We'll also use the ```-exec``` flag to execute our
command on each of find's results (which requires that we add a ```;```
to the end of our sed command).

```
find app/views/ -type f -exec sed -i '' -E 's/span([0-9])/col-md-\1/g'
```

Viola!  (V*oi*la?)

Cool.  But be really careful with this stuff.
