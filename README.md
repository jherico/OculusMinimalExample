OculusMinimalExample
==================

Example code for using the Oculus Rift

# Instructions for building (all platforms)

## Checking out 

	git clone https://github.com/jherico/OculusMinimalExample.git 

## Creating project files

	cd OculusMinimalExample
	mkdir build && cd build
	cmake ..  cmake .. -G "Visual Studio 12 2013 Win64"

Will show you a list of supported generators on your platform.

## Building

Open Visual Studio and open the `OculusMinimalExample.sln` solution in the from the `OculusMinimalExample/build` directory

Right click on the `OculusMinimalExample` and choose `Set as StartUp Project`. 

Press `F5` or click on the `Local Windows Debugger` toolbar button
