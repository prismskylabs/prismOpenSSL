OpenSSL
=======

Use this as an SSL submodule. It will contain binaries for each platform we support. The binaries in this submodule should, at all times, match the version of the source.

Source Files
------------
Source files are located in the src directory. If you update the source files to a later build of OpenSSL it is your responsibility to update the binaries for each platform along with it.

Using the Binaries
------------------
Binaries and platform specific files are located under the build/<platform> directories. For insantce, if you wanted to build against this submodule for windows, you would add build/windows/include to your include path and build/windows/lib to your library path.

Adding Another Platform
-----------------------
If you wish to add another platform simply use the Configure script in the source directory to configure OpenSSL for your target platform with the prefix pointing to build/<platform>. Once everything is configured to your satisfaction just make and make install and everything should show up in build/<platform> as expected. For example, this is how we built OpenSSL for the ISD platform:
<pre><code>cd src
./Configure -DOPENSSL_NO_HEARTBEATS shared --openssldir=../build/isd os/compiler:arm-linux-gnueabihf-
make
make install
</code></pre>
