# 🧠 Installing Cored

### How to install cored

Cored is a program that runs on the Coreum blockchain. You can install it in two ways:

* Use a prebuilt cored (only for Linux)
* Build cored from sources

<mark style="color:green;">**Use a prebuilt cored**</mark>

This is for Linux users who only use the command line.

Steps:

* Check your network variables are correct.
* Make a folder for cored.

```sh
mkdir -p $COREUM_HOME/bin
```

* Install curl, a tool for downloading files.
* Download cored and put it in the folder.

```sh
curl -LO https://github.com/CoreumFoundation/coreum/releases/download/$COREUM_VERSION/$COREUM_BINARY_NAME
mv $COREUM_BINARY_NAME $COREUM_HOME/bin/cored
```

If you get a 404 error then try the code below.

```sh
echo https://github.com/CoreumFoundation/coreum/releases/download/$COREUM_VERSION/$COREUM_BINARY_NAME
```

* Add cored to PATH and make it runnable.

```shell
export PATH=$PATH:$COREUM_HOME/bin
chmod +x $COREUM_HOME/bin/*
```

* Test cored with `cored version`.

You’re done! Go back to the previous Readme.&#x20;



**Build cored from sources**

Follow the [<mark style="color:green;">Build and Play guide</mark>](https://github.com/CoreumFoundation/coreum/blob/master/README.md#build-and-play) to make cored from the source code.

You’re done! Go back to the previous Readme.
