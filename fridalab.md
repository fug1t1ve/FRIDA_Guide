# FridaLab

Created by [Ross Marks](http://rossmarks.uk/blog/)

The app is similar to bomblabs it contains challenges suitable for a frida beginner to learn how the basics of frida works

[Download](https://rossmarks.uk/blog/fridalab/)


## Emulator
I prefer using [Genymotion(Windows 10 with virtualbox)](https://www.genymotion.com/download/) 

## Frida installation
- `pip install frida-tools`
- After installing frida-tools run the virtual anroid device 
- Then use `adb start-server` to start adb server then connect to the device `adb connect ip:port` (it automatically detects the device )
- You can see your device using `adb devices`
- Now Download Frida-server from [here](https://github.com/frida/frida/releases) for android (`frida-server-VER_NUM-android-x86.xz`)
- Unzip the server `unxz frida-server.xz`
- Then push ,set perms and run frida-server : 
    - `adb push frida-server /data/local/tmp/`
    - `adb shell "chmod 755 /data/local/tmp/frida-server"`
    - `adb shell "/data/local/tmp/frida-server &"`

## Getting Started
- Now your frida env is ready we can go ahead and install the apk and start tracing
- To install use `adb install fridalab.apk`
- To start tracing `frida -U -f uk.rossmarks.fridalab`
- Once you run the above commands you will get the frida shell: [img]
- Do `%resume` as it is debugging and will stop at "start" and the classes are not declared
- You can follow the above commands for any application you are debugging

## Decompiler
[jadx](https://github.com/skylot/jadx) - Dex to Java decompiler


## Challenges
- Challenge01: Change class challenge_01’s variable ‘chall01’ to 1
- Challenge02: Run chall02()
- Challenge03: Make chall03() return true
- Challenge04: Send “frida” to chall04()
- Challenge05: Always send “frida” to chall05()
- Challenge06: Run chall06() after 10 seconds with correct value
- Challenge07: Bruteforce check07Pin() then confirm with chall07()
- Challenge08: Change ‘check’ button’s text value to ‘Confirm’

## Writeup

Lets first decompile the apk using the jadx-gui(linked above).
After decompiling most of the source files that we need to analyse would be under the dir uk.rossmarks.fridalab it is the same for all other apks as well the main source files will be under the dir which you are using to trace in the frida command other are mostly resources or external library.Some of the cases would have some other dir in some CTF challs but that is rarely or related react api,that would be another blog.

### Challenge01:
![](https://i.imgur.com/oNkwnXu.png)
So after we decompile and enter the dir uk/rossmarks/fridalab we come across the challenge_01.java and as the challenge desc tells uss to change the  value of the variable declared in that class.
![](https://i.imgur.com/NeKkH5V.png)
To perform either an updation in java using frida we use `Java.perform` the syntax for this is:
```javascript=
Java.perform(function f(){
    "...type your functions you want to do here ...";
});
```
To use a class here as they are already initialised we can use `Java.use('class_path');` here for class challenge_01 we would do:
```javascript=
Java.perform(function f(){
    var chall1=Java.use('uk.rossmarks.fridalab.challenge_01');
});
```
We assign it to var as we need to update the component of the class we are basically calling the init() of challenge_01
now to update the var chall01:
```javascript=
Java.perform(function f(){
    var chall1=Java.use('uk.rossmarks.fridalab.challenge_01');
    chall1.chall01.value=1;
});
```
To update the value we have to use `.value`
After running this command in the frida shell we get:
![](https://i.imgur.com/X5uRK94.png)
remember dont put `\n` in frida shell it dosent accept that
![](https://i.imgur.com/gsQW6lo.png)

### Challenge02:
Can't see a different challenge02 class(java file) well check MainActivity it is named "Main activity" for some reason :shrug: 
Whenever a apk starts it intiate this class and calls the oncreate function.
![](https://i.imgur.com/8H7gyGK.png)
In the MainActivity class you can see the chall02() function so pretty easy right?
lets try the above thing just instead of modifying we are calling.
```javascript=
Java.perform(function f(){
    var chall2=Java.use('uk.rossmarks.fridalab.MainActivity');
    chall2.chall02();
});
```
should be enough ? lets check.
![](https://i.imgur.com/H4T02fi.png)
So, basically chall02() is an instance function and it cant be called as above method because we are not intiating a instance.
We need to have a running instance ,so lets create one.
```javascript=
var main;
Java.choose('uk.rossmarks.fridalab.MainActivity',{
  onMatch: function(instance) {
    main = instance;
  },
  onComplete: function() {}
});
main.chall02();
```
we are using Java.choose over here
**Note we will be using main variable instance for the later challenges.**
so lets perform the above things and check if we succeed.
![](https://i.imgur.com/TbQ2w21.png)
![](https://i.imgur.com/C5N5P83.png)

### Challenge03:
The chall03() is also there in MainActivity
![](https://i.imgur.com/tizTlut.png)
We need to overwite this function so that it returns true so there is a way we can do that
```javascript=
main.chall03.overload().implementaion=function(){
    return true;
}
```
![](https://i.imgur.com/eFpOJGt.png)

![](https://i.imgur.com/zjECgBr.png)

### Challenge04:
![](https://i.imgur.com/01EgpdU.png)
Since we already have our instance already intialised we just had to call the cahll04 with "frida"
```javascript=
main.chall04('frida')
```
![](https://i.imgur.com/8Ab2drR.png)
### Challenge05:
So in this we have to overload the function so that it calls the original function with "frida" as argument
![](https://i.imgur.com/RP7K0DB.png)

```javascript=
main.chall05.overload('java.lang.String').implementation=function (arg1){
    this.chall05.overload('java.lang.String').call(this,'frida');
    return;
}
```
![](https://i.imgur.com/G9IkPb7.png)

### Challenge06:
In this challenge we need to wait 10 seconds and then submit the correct value to the validation function
Our approach here is we overwriting the return value to true so taht whenever confirmchall06 is called it returns true
![](https://i.imgur.com/5QCqlJo.png)

```javascript=
Java.perform( function(){
	var chall6 = Java.use("uk.rossmarks.fridalab.challenge_06");
	chall6.confirmChall06.implementation = function(a) {
        var returnValue = this.confirmChall06(a);
        var newReturnValue = true;
        return newReturnValue;
	};
	var main_instance;
	Java.choose("uk.rossmarks.fridalab.MainActivity" , {
  		onMatch : function(instance){
    			instance.chall06(1);
			main_instance=instance;
  		},
  	onComplete:function(){}
	});
});
```
![](https://i.imgur.com/IGBAUQU.png)

### Challenge07:
There are 2 ways to solve it
the first way is we can just look at the value of chall07 that it the pin
```javascript=
var chall07=Java.use("uk.rossmarks.fridalab.challenge_07");
console.log(chall07.chall07.value);
main.chall07('5105');
```
![](https://i.imgur.com/2e8thFZ.png)

The other way is bruteforce the check07pin
```javascript=
function pad(num, size) {
    num = num.toString();
    while (num.length < size) num = "0" + num;
    return num;
}
for(var i=9999;i>=0;i--){
  if (chall07.check07Pin()) {
    main.chall07(pad(i,4));
    break;
  }
}
```
pad(i,4) converts i=0 to 0000 and returns as string
![](https://i.imgur.com/7WXejmO.png)

![](https://i.imgur.com/hEFBlzO.png)

### Challenge08:
This Challenge is different from other challenges as it deals with modifying the UI of the apk and as we can see in check for chall08 we can find that its passing the ui component id as `R.id.check` which if see in decompilation of r class is:
![](https://i.imgur.com/AqLtq8S.png)

So now that we know the id of the component we can then cast the button on an operator by using `Java.cast()` and then change text using `String.setText()`
```javascript=
var button = Java.use('android.widget.Button');
var check = main.findViewById(0x7f07002f);
var confirm = Java.cast(checkid, button);
var string = Java.use('java.lang.String');
confirm.setText(string.$new("Confirm"));
```
![](https://i.imgur.com/LrPcQbH.png)

## Conclusion
This set of challenges were pretty good and was a good introduction to a begineer in android reversing as it not only covers the frida part but also the basic structure of an apk, There are many more set of challenges like fridalab,and soon i might try to make one which might feature in the inctfi(no promises) :stuck_out_tongue: 

Thanks to the author [Ross Marks](http://rossmarks.uk/blog/) for such a cool set of challenges

