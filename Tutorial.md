![[Pasted image 20231204155641.png]]

Would you like to be able to render this monstrosity above me? Of course you would! With this hastily-written tutorial, you too can follow your dreams.

```
Prerequisite
Part 0 - Setup
Part 0.5 - Helper Class and Maths Functions
Part 1 - Models
	- 1.1: Textures
	- 1.2: Meshes
	- 1.3: Shaders
	- 1.4: Blender Tutorial
Part 1.5 - Testing
Part 2 - Camera
	- 2.1: Theory
		- 2.1.1: Delta Time
		- 2.1.2: Euler Angles - Pitch, Yaw, and Roll
	- 2.2: Camera Class
Part 3 - Lighting
Credit
```

## Prerequisite
 Contains:
 - Models
	 - [x] Texture loading
	 - [x] Mesh loading
	 - [x] Shaders
	 - [x] Simple Blender tutorial
 - Camera movement
	 - [x] View and space positioning
 - Lighting
	 - [x] Spot, point, and directional
	 - [x] Arbitrary count of lights

This tutorial assumes you're working with:
- Windows 10/11
- Visual Studio 2017+
- C++ $\ge14$
- freeGLUT
- GLEW
- GLM (optional)
- Assimp
- stb_image.h
- Blender

This tutorial also assumes you're already familiar with VBOs and VAOs. If not, read LearnOpenGL's or OGLDEV's tutorials on the matter.
Also, you can use https://docs.gl/ to look up any OpenGL commands you're unfamiliar with. Don't forget that we're using version 3.3 of OpenGL.

***N.B.***
There will be no/minimal comments in this code so you are encouraged to write them yourself. This tutorial is intended to function as a guide, not free code, although full entries of the code used will be available at the end of each section. Use them wisely; plagiarism is a very serious offense.

## Part 0 - Setup
This project uses 64-bit (x64) versions of libraries.

Create a new C++ console project and make sure to tick "Place solution and project in the same directory" if it isn't already. Download freeGLUT, GLEW, GLM (optional), Assimp, and stbi.h from:
- GLM: https://github.com/g-truc/glm/tags
- freeGLUT: https://github.com/FreeGLUTProject/freeglut
- GLEW: https://github.com/nigels-com/glew
- stb_image.h: https://github.com/nothings/stb/blob/master/stb_image.h (just download the file)
- Assimp
	- https://github.com/assimp/assimp/releases/tag/v5.3.1
	- https://kimkulling.itch.io/the-asset-importer-lib (pre-compiled binary, x64)

Alternatively, use the libraries in the zipped folder labelled "Libraries". Note that these versions may be outdated, but they are the ones used in the project. Install at your own risk. I didn't put a virus in there, I promise.

To install the x64 version of Assimp yourself, go to the second link above and download the .exe file. Follow the wizard and fully install the files. The download folder can be placed wherever you like initially, but you should copy or move the new folder into your Visual Studio project's base directory (if you plan on exporting your files).

### Project Setup
- Copy everything inside Libraries to the base directory of the current project
- In the project Property Pages ([Menu Bar] Project/[ProjectName] Properties), set "Configuration Properties -> Debugging -> Working Directory" to "$(OutDir)"
- Add the following lines to your properties:
	- `C/C++ -> General -> Additional Include Directories` = `$(SolutionDir)Assimp\include;$(SolutionDir)freeglut\include;$(SolutionDir)glew-1.10.0\include;$(SolutionDir)glm`
	- `Linker -> General -> Additional Library Directories` = `$(SolutionDir)Assimp\lib\x64;$(SolutionDir)freeglut\lib\x64;$(SolutionDir)glew-1.10.0\lib\Release\x64`
	- `Linker -> Input -> Additional Dependencies` = `$(CoreLibraryDependencies);%(AdditionalDependencies);freeglut.lib;glew32.lib;assimp-vc143-mt.lib`
- In the menu just below the Menu Bar, set the project type to Release and make sure the value next to it is "x64". Build using Ctrl+B, then switch project type to Debug and enter Ctrl+B again (the order of Release and Debug don't matter; just make sure you do both).
- Copy the x64 .dll files for each eligible library into the Debug and Release folders. These files are in Libraries/DLLs, or can be found at:
	- freeglut: `freeglut/bin/x64/freeglut.dll`
	- glew: `glew-1.10.0/bin/Release/x64/glew32.dll` (don't mind the file name)
	- Assimp: `Assimp/bin/x64/assimp-vc143-mt.dll`

You can rebuild the project. Try running the `main.cpp` file so check that it works. Everything should be working as intended. If not, make sure the above paths are correct. Try to find the files yourself and see if they are where they should be.

#### Organisation and Hierarchy
Something important to do at this point is to setup your directory. Specifically, you should begin to setup the hierarchy you will use to store models and textures. This isn't necessary, but it allows you to avoid placing duplicates of your files in your x64/Debug and x64/Release folders. For any file you want to access, be it .txt, .obj, .glsl, or otherwise, I recommend placing this file in a folder in the base repository. 

For example, if I wanted somewhere safe to store my shaders, I might put it in the Shaders folder in the base directory. In my case, I have a folder called "Models" which has folders within it labelled with the title of the model I want to use. This folder holds my .obj and .mtl files (as well as my .blend file from Blender if it exists) and another folder called "Textures" which, as you may have guessed, holds my textures. For some objects called "test_cube" and "spider", the hierarchy looks like this:
```
Project Directory/ 
	- Assimp/
		- ...
	- freeglut/
		- ...
	- glew-1.10.0/
		- ...
	- glm/
		- ...
	- x64/
		- Debug/
			- ...
		- Release/
			- ...
	- Shaders/
		- vertex.glsl
		- fragment.glsl
		- cubemap_vert.glsl
		- cubemap_frag.glsl
	- Models/
		- test_cube/
			- test_cube.obj
			- test_cube.mtl
			- test_cube.blend
			- Textures/
				- face1.jpg
				- face2.jpg
		- spider/
			- spider.obj
			- spider.mtl
			- Textures/
				- spider_leg.jpg
				- spider_body.jpg
				- spider_eye.jpg
	- main.cpp
	- main.h
	- mesh.cpp
	- mesh.h
	- ...
	- stb_image.h
	- ...
```
You can of course model your hierarchy in any way you like, but this is the way I did mine. Keeping the textures in a subfolder within the model folder seemed best for organisation and it allows me to find the model folder and files with only the model's name.


### Bugs
#### material.inl
`...\assimp\include\assimp\material.inl(101,47): error C2589: '(': illegal token on right side of '::'`
This bug can be fixed by changing the line
```cpp
iNum = static_cast<unsigned int>(std::min(static_cast<size_t>(iNum), prop->mDataLength / sizeof(Type)));
```
to
```cpp
iNum = static_cast<unsigned int>((std::min)(static_cast<size_t>(iNum), prop->mDataLength / sizeof(Type)));
```
The line may not be on exactly line 101. Search for the error line above and replace it.

#### Some parts of my code aren't running
Check that you're not running your code in Release mode. Release mode ignores `assert` statements, so if a function that returns a boolean is ran inside an `assert` statement, this code will be skipped if you are running in Release mode. Try storing the output to a variable and asserting that, so that your code still runs.

## Part 0.5 - Helper Class and Maths Functions
I use a helper class to store various things I don't want to redefine across classes. The class can then be imported into multiple other classes to be reused at will. I also have a Scene Manager class I use to keep track of variable data across the entire project such as [[#2.1.1 Delta Time|delta time]]. This is known as the [singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern). It isn't fully implemented (no destructors because I don't know how to make them) but you can make one yourself pretty easily, the same way you'd make any other class. Just make everything static and you're good to go :3

### Help.h
(Still not sure I use everything here...)
```cpp
#pragma once
#include <map>
#include <vector> // STL dynamic memory.
#include <string>
#include <fstream>
#include <iostream>
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include "maths_funcs.h"
class Help
{
public:
	static std::string readFile(const char* path);
	static float wrap(float val, float min, float max);
	static float clamp(float val, float min, float max);
	static float deg2Rad(float val);
	static float rad2Deg(float val);
	
	static float delta;
	static const int width = 800;
	static const int height = 600;
};
```

### Help.cpp
```cpp
#include "Help.h"

float Help::delta = 0.0f;

std::string Help::readFile(const char* path) {
	std::ifstream file(path);
	if (!file.is_open()) {
		std::cout << "Failed to open file." << std::endl;
		return "";
	}

	std::string text, line;
	while (getline(file, line)) {
		text += line + "\n";
	}

	file.close();
	text.append("\0");
	return text;
}

// Wrap a value between min and max. If val is greater than max, it will wraparound to min and begin climbing from there, and vice versa.
float Help::wrap(float val, float min, float max) {
	return fmod(min + (val - min), max - min);
}

// Clamp a value between a minimum and maximum so that val cannot be greater than max or smaller than min.
float Help::clamp(float val, float min, float max) {
	if (val < min) val = min;
	if (val > max) val = max;
	return val;
}

// Convert degrees to radians
float Help::deg2Rad(float val) {
	return val * ONE_DEG_IN_RAD;
}

// Convert radians to degrees
float Help::rad2Deg(float val) {
	return val * ONE_RAD_IN_DEG;
}
```

Additionally, I modified the `maths_funcs.cpp` file to add a bit more functionality to some data types there. You could decide to ignore the file and just use GLM functions directly, but I was bored, so this exists now. The file will be included outside this document. The changes made were:
##### maths_funcs.h
```cpp
// ...
struct vec3 {
	// ...
	//! create from 1 scalars (ADDED BY IRIS)
	vec3(float x);
	//! multiply with vector (ADDED BY IRIS)
	vec3& operator* (const vec3& rhs);
	//! check equality of vectors (ADDED BY IRIS)
	bool operator== (const vec3& rhs);
	//! check inequality of vectors (ADDED BY IRIS)
	bool operator!= (const vec3& rhs);
};
```

##### maths_funcs.cpp
```cpp
// ADDED BY IRIS
vec3::vec3(float x) {
	v[0] = x;
	v[1] = x;
	v[2] = x;
}

// ADDED BY IRIS
vec3& vec3::operator* (const vec3& rhs) {
	v[0] *= rhs.v[0];
	v[1] *= rhs.v[1];
	v[2] *= rhs.v[2];
	return *this;
}

// ADDED BY IRIS
bool vec3::operator== (const vec3& rhs) {
	return
		v[0] == rhs.v[0] &&
		v[1] == rhs.v[1] &&
		v[2] == rhs.v[2];
}

// ADDED BY IRIS
bool vec3::operator!= (const vec3& rhs) {
	return
		v[0] != rhs.v[0] ||
		v[1] != rhs.v[1] ||
		v[2] != rhs.v[2];
}
```

## Part 1 - Models
This part of the tutorial uses OGLDEV's mesh and texture classes. You can find the version I used here: https://github.com/emeiri/ogldev/tree/master/tutorial33
### 1.1: Textures
For all intents and purposes, a texture is simply an image we map onto a mesh. We can use any such image we please, but we need to get enough information about it to pass it into OpenGL's various macros. We will be using solely .jpg in this tutorial, so be mindful if other format don't work.

We'll start with a Texture class file. You can add a new class in Visual Studio by right-clicking the name of the project in the Solution Explorer on the right hand side of the window and going to `Add -> Class...`. There's no difference between .h and .hpp files, so take your pick.
![[Pasted image 20231204174709.png|500]]
#### Header (.h/.hpp)
In the Texture.h file, note the `#pragma once` at the top. This is a compiler instruction that tells GCC not to include this header file more than once. An alternative representation of it is 
```cpp
#ifndef TEXTURE_H
#define TEXTURE_H
```
I recommend putting/leaving `#pragma once` at the top of all your header files if it isn't there already.

Add the following imports:
```cpp
#include <string>
#include <iostream>
#include <GL/glew.h>
```
You can choose to use another string representation like `const char*` but `std::string` is easier in my opinion.

Next is the class body. [The code this is taken from](https://github.com/emeiri/ogldev/blob/master/Include/ogldev_texture.h) has more features, but we're only taking the ones we need for now (you can take more if you want, I can't stop you). We want the following body:
```cpp
class Texture
{
public:
	Texture(const std::string, GLenum);
	Texture(GLenum);
	bool load();
	bool load(unsigned int, void*);
	void bind(GLenum);

	std::string file_name;
	GLenum textureEnum;
	unsigned int texture;
	int _width, _height, _nrChannels, _bits_per_pixel;
};
```
- `Texture(const std::string, GLenum)` takes in a string of its path and the type of texture it will be
- `Texture(GLenum)` takes in only a type of texture and is to be used for preparing to load in textured embedded in the mesh file
- `load()` will load in the texture we give it using the first class declaration
- `load(unsigned int, void*)` takes in a buffer size and binary image data to load in an embedded texture. The unsigned int `texture` will be a buffer object to hold information about this particular texture.
- `bind(GLenum)` binds the texture and prepares it for reading

The `GLenum` we get from instantiation and `bind` are given to the `textureEnum` variable, and the `file_name` is similar. `texture` is our buffer for the texture. `_width` and `_height` are the width and height of the image, respectively, `_nrChannels` is how many colour channels the texture has, and `_bits_per_pixel` is the number of bits in our texture, which determines the type of texture it may be. (I'm pretty sure `_nrChannels` and `_bits_per_pixel` refer to roughly the same thing, but they are used in different contexts here).

Also, note that we don't actually have to give function declaration parameters names in the header file. You can give it a name or not; it won't change anything.

#### Implementation (.cpp)
In your Texture.h (or .hpp) file, add the following line just below the other import:
```cpp
#include "stb_image.h"
#define STB_IMAGE_IMPLEMENTATION
```
This is necessary for reasons I don't understand but the developers tell us is very important.

To start, we can define the `Texture()` function as follows:
```cpp
Texture::Texture(const std::string fname, GLenum texType = GL_TEXTURE_2D) {
	textureEnum = texType;
	file_name = fname;
}
```
We will need a reference to the path specified in the `fname` parameter (it does need to be named here) when we later load the texture, as well as the `texType` enum. We default to `GL_TEXTURE_2D`.

```cpp
Texture::Texture(GLenum texType = GL_TEXTURE_2D) {
	textureEnum = texType;
}
```
The second class declaration is even simpler.

Next, we can define the `bind` function:
```cpp
void Texture::bind(GLenum textureUnit) {
	glActiveTexture(textureUnit);
	glBindTexture(GL_TEXTURE_2D, texture);
}
```
Also a very short function. `glActiveTexture` sets the current texture to the active texture for this mesh. `glBindTexture` then binds it to the buffer `texture` that we defined in the header file, specifying it as a `GL_TEXTURE_2D`.

Finally, we can write the `load` functions.
##### `load`
We start by generating a buffer for the texture at the address our stored buffer object:
```cpp
glGenTextures(1, &texture);
```
The `1` specifies that we're only generating one texture name.

Next, we add:
```cpp
int _width, _height, nrChannels;
stbi_set_flip_vertically_on_load(true);
unsigned char* data = stbi_load(file_name.c_str(), &_width, &_height, &nrChannels, 0);
```
The first three variables are more buffer objects to store the width, height, and number of colour channels of the image, respectively. We set `stbi_set_flip_vertically_on_load(true)` to flip the image to align it with OpenGL's texture renderer. (This may conflict with a step at a later stage, but you should leave this one in first.) The `data` variable loads the texture from the file name we saved and loads information into the aforementioned buffers.

The main body of this function is:
```cpp
if (data)
{
	if (textureEnum == GL_TEXTURE_2D) {
		glBindTexture(textureEnum, texture);
		glTexImage2D(textureEnum, 0, GL_RGB, _width, _height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
		glGenerateMipmap(textureEnum);
		
		// set the texture wrapping/filtering options (on the currently bound texture object)
		glTexParameteri(textureEnum, GL_TEXTURE_WRAP_S, GL_REPEAT);
		glTexParameteri(textureEnum, GL_TEXTURE_WRAP_T, GL_REPEAT);
		glTexParameteri(textureEnum, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
		glTexParameteri(textureEnum, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	}
	else {
		printf("Texture type %x is not supported.", textureEnum);
		exit(1);
	}
}
else
{
	std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
return true;
```

To start, we check if the data we loaded actually contains any information. If not, we simply don't load it. If it does, we check if the texture type is of type `GL_TEXTURE_2D`, which it is, because we manually defined it so (you can cut this conditional out if you want; I probably will too). If this passes (again, it will), we bind the texture buffer with our chosen texture type. We then create an image using the buffers we defined when loading it in. We also generate mipmaps for the image.

Now, we set texture parameters for wrapping and filtering modes. Here, `S` means a u-coordinate (which means an x-coordinate) and `T` means a v-coordinate (y). We want to set the texture to repeat if it goes outside the (0,0) to (1,1) range (if it's too small, basically) and to set the minifying and magnifying functions to use linear interpolation when scaling (read your notes for that lol).

Finally, we free the data since we aren't using it anymore and return true, because why not?

##### `load(unsigned int, void*)`
The second `load` function is very similar to the first, but things are done slightly out of order. To start, our `data` variable is created with:
```cpp
void* data = stbi_load_from_memory((const stbi_uc*)img_data, buffer, &_width, &_height, &_bits_per_pixel, 0);
```
where `img_data` is the data.

We no longer need to check if the data is fine because we'll check if this data exists before calling this function, but we do need to check the type of texture it is to figure out how to load it. We can use a simple switch statement on `_bits_per_pixel` to find out the type of texture we have:
```cpp
switch (_bits_per_pixel)
{
case 1:
	glTexImage2D(textureEnum, 0, GL_RED, _width, _height, 0, GL_RED, GL_UNSIGNED_BYTE, data);
	break;
case 3:
	glTexImage2D(textureEnum, 0, GL_RGB, _width, _height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
	break;
case 4:
	glTexImage2D(textureEnum, 0, GL_RGBA, _width, _height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
	break;
default:
	printf("not supported");
	break;
}
```
This switch statement loads in only red channels if it only has 1, RGB channels if it has 3, and RGB with alpha (transparency) channels if it has 4. Any other number would be weird so we don't support it. This check should also be done for the other `load()` function, we I skipped it here.

Finally, we set texture parameters like normal:
```cpp
glTexParameteri(textureEnum, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(textureEnum, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(textureEnum, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(textureEnum, GL_TEXTURE_WRAP_T, GL_REPEAT);
return true;
```

#### Full File
##### Texture.h
```cpp
#pragma once

#include <string>
#include <iostream>
#include <GL/glew.h>

class Texture
{
public:
	Texture(const std::string, GLenum);
	bool load();
	void bind(GLenum);

	std::string file_name;
	GLenum textureEnum;
	unsigned int texture;
};
```

##### Texture.cpp
```cpp
#include "Texture.h"
#include "stb_image.h"
#define STB_IMAGE_IMPLEMENTATION

Texture::Texture(GLenum textureType = GL_TEXTURE_2D) {
	textureEnum = textureType;
}

Texture::Texture(const std::string fname, GLenum textureType = GL_TEXTURE_2D) {
	textureEnum = textureType;
	file_name = fname;
}

void Texture::bind(GLenum textureUnit) {
	glActiveTexture(textureUnit);
	glBindTexture(GL_TEXTURE_2D, texture);
}


bool Texture::load() {
	glGenTextures(1, &texture);

	int _width, _height, nrChannels;
	stbi_set_flip_vertically_on_load(true);
	unsigned char* data = stbi_load(file_name.c_str(), &_width, &_height, &nrChannels, 0);
	if (data)
	{
		if (textureEnum == GL_TEXTURE_2D) {
			glBindTexture(textureEnum, texture);
			glTexImage2D(textureEnum, 0, GL_RGB, _width, _height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
			glGenerateMipmap(textureEnum);
			
			glTexParameteri(textureEnum, GL_TEXTURE_WRAP_S, GL_REPEAT);
			glTexParameteri(textureEnum, GL_TEXTURE_WRAP_T, GL_REPEAT);
			glTexParameteri(textureEnum, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
			glTexParameteri(textureEnum, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
		}
		else {
			printf("Texture type %x is not supported.", textureEnum);
			exit(1);
		}
	}
	else
	{
		std::cout << "Failed to load texture" << std::endl;
	}
	stbi_image_free(data);
	return true;
}


bool Texture::load(unsigned int buffer, void* img_data) {
	void* data = stbi_load_from_memory((const stbi_uc*)img_data, buffer, &_width, &_height, &_bits_per_pixel, 0);
	stbi_set_flip_vertically_on_load(true);
	glGenTextures(1, &texture);
	glBindTexture(textureEnum, texture);

	switch (_bits_per_pixel)
	{
	case 1:
		glTexImage2D(textureEnum, 0, GL_RED, _width, _height, 0, GL_RED, GL_UNSIGNED_BYTE, data);
		break;
	case 3:
		glTexImage2D(textureEnum, 0, GL_RGB, _width, _height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
		break;
	case 4:
		glTexImage2D(textureEnum, 0, GL_RGBA, _width, _height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
		break;
	default:
		printf("not supported");
		break;
	}

	glTexParameteri(textureEnum, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(textureEnum, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(textureEnum, GL_TEXTURE_WRAP_S, GL_REPEAT);
	glTexParameteri(textureEnum, GL_TEXTURE_WRAP_T, GL_REPEAT);
	return true;
}
```

### 1.2: Meshes
There's a bit of confusion about whether "model" or "mesh" is the correct term to use in any given situation. Personally, I don't bother thinking about it, but I've found that people tend to use the following definitions:
- sub-mesh: a group of vertices, indices, and normals used as part of a mesh
- mesh: a group of sub-meshes used to define an object in space
- model: a group of meshes and textures used to define a fully rendered object

The Mesh class would more closely follow the definition of model here, but it's still called this way because the tutorial I followed called it this and I didn't want to get confused. I use both model and mesh interchangeably in this tutorial, and sub-mesh to clarify certain terms.
#### Header (Mesh.h/.hpp)
We'll start with the Mesh.h file.  We wrote the Texture class first because we may need several Texture objects for each mesh. We'll need access to the file system as well as a few other things, so we'll use this list of imports:
```cpp
#include <filesystem>
#include <string>
#include <map>
#include <stdio.h>
#include <stddef.h>
#include <math.h>
#include <vector>
#include <assert.h>

#include <assimp/cimport.h>
#include <assimp/Importer.hpp>
#include <assimp/scene.h>
#include <assimp/postprocess.h>

#include <GL/glew.h>
#include <GL/freeglut.h>

#include "Help.h"
#include "Texture.h"
```
This is a very long list and I'm not sure I actually use them all, but you can knock yourself out.

I defined a macro to store the load flags of Assimp's importer, which we use later. We will go over them when we get to them, but I wrote them like:
```cpp
#define ASSIMP_LOAD_FLAGS aiProcess_Triangulate | aiProcess_PreTransformVertices
```

Next, we have the body of the header file. We start with a `struct MeshObject`, which stores information about each individual mesh within this Mesh object.
```cpp
struct MeshObject {
	MeshObject() {
		n_Indices = 0;
		baseVertex = 0;
		baseIndex = 0;
		materialIndex = 0;
	}
	unsigned int n_Indices;
	unsigned int baseVertex;
	unsigned int baseIndex;
	unsigned int materialIndex;
};
```

Now, we declare our functions:
```cpp
Mesh() { ; }
Mesh(std::string mesh_name) { loadMesh(mesh_name); }
bool loadMesh(std::string mesh_name);
bool initScene(const aiScene*, std::string);
void initSingleMesh(const aiMesh*);
bool initMaterials(const aiScene*, std::string);
void populateBuffers();
void render(unsigned int, const mat4*);
```

We also have more macros, this time to define locations of certain variables in our vertex shader. We will talk about these more when we get to shaders.
```cpp
#define POSITION_VBO 0
#define NORMAL_VBO 1
#define TEXTURE_VBO 2
#define INSTANCE_VBO 3
```

Finally, our member variables:
```cpp
unsigned int VAO; // mesh vao
unsigned int p_VBO; // position vbo
unsigned int n_VBO; // normal vbo
unsigned int t_VBO; // texture vbo
unsigned int EBO; // index (element) vbo (ebo)
unsigned int Instance; // instance vbo (ibo)

std::vector<vec3> m_Positions; // position vectors
std::vector<vec3> m_Normals; // normal vectors
std::vector<vec2> m_TexCoords; // texture coordinate vectors
std::vector<MeshObject> m_Meshes; // mesh objects
std::vector<Texture*> m_Textures; // texture objects
std::vector<unsigned int> m_Indices; // index locations
```

#### Implementation (Mesh.cpp)
##### loadMesh
We'll start with the `loadMesh` function. This function takes in the path to your model and uses Assimp to load it. For the purposes of this tutorial, I will be using .obj files exported by Blender. Note the similarities between this function and `load_mesh` in the Lab 3 code.
```cpp
bool Mesh::loadMesh(std::string file_name) {
	glGenVertexArrays(1, &VAO);
	glBindVertexArray(VAO);
	glGenBuffers(1, &p_VBO);
	glGenBuffers(1, &n_VBO);
	glGenBuffers(1, &t_VBO);
	glGenBuffers(1, &EBO);
	glGenBuffers(1, &Instance);
	
	std::string rpath = MODELDIR(file_name) + file_name;
	const aiScene* scene = aiImportFile(
		rpath.c_str(),
		ASSIMP_LOAD_FLAGS
	);

	bool valid_scene = false;

	if (!scene) {
		fprintf(stderr, "ERROR: reading mesh %s\n%s", rpath.c_str(), aiGetErrorString());
		valid_scene = false;
	}
	else {
		valid_scene = initScene(scene, file_name);
	}

	glBindVertexArray(0);
	return valid_scene;
}
```
We start by generating vertex arrays for our VAO and binding it. Then, we generate buffers for our VBOs, indices, and instances.

We then load the model we want to use. I use another macro here, `MODELDIR`, which gets the model directory so I don't have to specify it myself. (This is why my folder names are the same as the model object :3) Since I'm running this on 64-bit (x64), my macro looks like:
```
#define MODELDIR(m) "../../Models/" + m.substr(0, m.find(".")) + "/"
```
If you're on 32-bit (Win32), you can omit the first "``../``" of that string. 

Next, we get the Assimp scene object by importing the model using the path we got. This is where we use the `ASSIMP_LOAD_FLAGS` macro I talked about earlier.
These load flags tell Assimp how it should import the model. There are several flags available to us, but we only really need to use 2 (or 3).
- `aiProcess_Triangulate`: this flag triangulates the mesh to better fit with OpenGL's renderer.
- `aiProcess_PreTransformVertices`: this flag transforms the vertices in the mesh before rendering. It's necessary if you wish to use scaling or rotation with your mesh.
- `aiProcess_FlipUVs`: this flag will flip the UV coordinates of the mesh. If your textures are upside-down, try this flag out to see if it fixes it. It's omitted here because of a potential clash with the `stbi_set_flip_vertically_on_load()` function we used when importing the texture, but you can mess with both as much as you like.

We then check to make sure the scene loaded our mesh successfully. If not, it will print it to the console for our sakes. If it does load, we can then initiate our scene. Afterwards, we bind the VAO to 0 to avoid accidentally modifying it when loading the mesh.

##### initScene
```cpp
bool Mesh::initScene(const aiScene* scene, std::string file_name) {
	m_Meshes.resize(scene->mNumMeshes);
	m_Textures.resize(scene->mNumMaterials);

	unsigned int nvertices = 0;
	unsigned int nindices = 0;

	// Count all vertices and indices
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		m_Meshes[i].materialIndex = scene->mMeshes[i]->mMaterialIndex;
		m_Meshes[i].n_Indices = scene->mMeshes[i]->mNumFaces * 3;
		m_Meshes[i].baseVertex = nvertices;
		m_Meshes[i].baseIndex = nindices;

		// Move forward by the corresponding number of vertices/indices to find the base of the next vertex/index
		nvertices += scene->mMeshes[i]->mNumVertices;
		nindices += m_Meshes[i].n_Indices;
	}

	// Reallocate space for structure of arrays (SOA) values
	m_Positions.reserve(nvertices);
	m_Normals.reserve(nvertices);
	m_TexCoords.reserve(nvertices);
	m_Indices.reserve(nindices);

	// Initialise meshes
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		const aiMesh* am = scene->mMeshes[i];
		initSingleMesh(am);
	}

	if (!initMaterials(scene, file_name)) {
		return false;
	}

	populateBuffers();
	return glGetError() == GL_NO_ERROR;
}
```

This function initialises the scene. It takes in the scene object from `loadMesh` and the same path to the mesh. 

We resize the meshes and textures arrays with the number of meshes and textures in the scene we generated. We then count the number of vertices and indices for each sub-mesh, as well as calculate the base vertex and index for each sub-mesh. This base will be the starting point for each new sub-mesh to begin building off of. Using data this way is known as "[Structure of Arrays](https://en.wikipedia.org/wiki/AoS_and_SoA)". You can find out more about it here: https://youtu.be/sP_kiODC25Q?feature=shared&t=545. Note that this way of doing it may change later down the line as we get to skeletal animations, but I haven't gotten there yet so I have no idea how it might change :3

We then initialise each mesh using `initSingleMesh`, and then initialise materials for the entire model using `initMaterials`. Finally, we populate the vertex buffers, index buffer, and instance buffer using `populateBuffers`.

##### initSingleMesh
```cpp
void Mesh::initSingleMesh(const aiMesh* amesh) {
	const aiVector3D Zero3D(0.0f, 0.0f, 0.0f);

	// Populate the vertex attribute vectors
	for (unsigned int i = 0; i < amesh->mNumVertices; i++) {
		const aiVector3D& pPos = amesh->mVertices[i];
		const aiVector3D& pNormal = amesh->mNormals[i];
		const aiVector3D& pTexCoord = amesh->HasTextureCoords(0) ? amesh->mTextureCoords[0][i] : Zero3D;

		m_Positions.push_back(vec3(pPos.x, pPos.y, pPos.z));
		m_Normals.push_back(vec3(pNormal.x, pNormal.y, pNormal.z));
		m_TexCoords.push_back(vec2(pTexCoord.x, pTexCoord.y));
	}

	// Populate the index buffer
	for (unsigned int i = 0; i < amesh->mNumFaces; i++) {
		const aiFace& Face = amesh->mFaces[i];
		assert(Face.mNumIndices == 3);
		m_Indices.push_back(Face.mIndices[0]);
		m_Indices.push_back(Face.mIndices[1]);
		m_Indices.push_back(Face.mIndices[2]);
	}
}
```

Here, we create an `aiVector3D` filled with zeros. Then we populate the vertex attribute with vectors. We get the address of each member variable of the `aiMesh` parameter we took in and add it to our own arrays. We do this for both the vertices and the indices. It's very similar to `initScene()`, simply looping over several arrays and adding them to our own.

##### initMaterials
```cpp
bool Mesh::initMaterials(const aiScene* scene, std::string file_name) {
	bool valid = true;
	std::string dir = MODELDIR(file_name);
	for (unsigned int i = 0; i < scene->mNumMaterials; i++) {
		const aiMaterial* pMaterial = scene->mMaterials[i];
		m_Textures[i] = NULL;

		if (pMaterial->GetTextureCount(aiTextureType_DIFFUSE) > 0) {
			aiString Path;

			if (pMaterial->GetTexture(aiTextureType_DIFFUSE, 0, &Path, NULL, NULL, NULL, NULL, NULL) == AI_SUCCESS) {
				const aiTexture* cTex = scene->GetEmbeddedTexture(Path.C_Str());
				if (cTex) {
					printf("Loaded embedded texture type %s\n", cTex->achFormatHint);
					m_Materials[index].diffTex = new Texture(GL_TEXTURE_2D);
					unsigned int buffer = cTex->mWidth;
					m_Materials[index].diffTex->load(buffer, cTex->pcData);
				} else {
					std::string p(Path.data);
					std::cout << p << std::endl;
					if (p.substr(0, 2) == ".\\") {
						p = p.substr(2, p.size() - 2);
					}
					std::string fullPath = dir + p;
					m_Materials[index].diffTex = new Texture(fullPath, GL_TEXTURE_2D);
					if (!m_Materials[index].diffTex->load()) {
						printf("Error loading diffuse texture '%s'\n", fullPath.c_str());
					} else {
						printf("Loaded texture '%s'\n", fullPath.c_str());
					}
				}
			}
		}
	}
	return valid;
}
```

Here, we initialise the materials to be used in our texture. 

If the texture is embedded, we first print out the type of texture format it was stored in. If it was a jpg, this would print `Loaded embedded texture type jpg`. Next, we create a new Texture of type `GL_TEXTURE_2D` and a buffer to store the texture's width (in size, not its actual dimensions). Then, we use the second type of `Texture.load()` to load an embedded texture.

If the texture is external, we use the path to our file to look for the directory the textures are stored in. Then, we iterate over each material in the scene and find the path to it. .obj files use .mtl files to store the location of textures they use. Since ours are stored near our model files, we don't need to search too far to find them. 

`Path.data` returns the path to these texture files. We access this path and use
```
if (p.substr(0, 2) == ".\\") {
	p = p.substr(2, p.size() - 2);
}
```
to get the final directory the file is located in. For example, if the path to the texture is `"C:\\Users\\Iris\\Desktop\\Tutorial\\Models\\cube\\Texture\\egg.jpg"`, `p` would store `"egg.jpg"`.

Then we create a new Texture object and load it using this new path and specify that it uses `GL_TEXTURE_2D`. We load this Texture object and if it fails, delete it and note that at least one texture failed to load.

##### populateBuffers
```cpp
void Mesh::populateBuffers() {
	glBindBuffer(GL_ARRAY_BUFFER, p_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_Positions[0]) * m_Positions.size(), &m_Positions[0], GL_STATIC_DRAW);
	glVertexAttribPointer(POSITION_VBO, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(POSITION_VBO);

	glBindBuffer(GL_ARRAY_BUFFER, n_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_Normals[0]) * m_Normals.size(), &m_Normals[0], GL_STATIC_DRAW);
	glVertexAttribPointer(NORMAL_VBO, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(NORMAL_VBO);

	glBindBuffer(GL_ARRAY_BUFFER, t_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_TexCoords[0]) * m_TexCoords.size(), &m_TexCoords[0], GL_STATIC_DRAW);
	glVertexAttribPointer(TEXTURE_VBO, 2, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(TEXTURE_VBO);

	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(m_Indices[0]) * m_Indices.size(), &m_Indices[0], GL_STATIC_DRAW);

	glBindBuffer(GL_ARRAY_BUFFER, IBO);
	for (unsigned int i = 0; i < 4; i++) {
		glEnableVertexAttribArray(INSTANCE_VBO + i);
		glVertexAttribPointer(INSTANCE_VBO + i, 4, GL_FLOAT, GL_FALSE, sizeof(mat4), (const void*)(i * sizeof(vec4)));
		glVertexAttribDivisor(INSTANCE_VBO + i, 1); // tell OpenGL this is an instanced vertex attribute.
	}
}
```

In this function, we bind, load, and enable each VBO, EBO, and instance buffer object we have. For the first VBO (the position vertex), we bind `p_VBO` to tell the compiler that we currently wish to modify it. We tell it that we want to create an array buffer with `m_Positions.size()` 3D vectors (we used `vec3` for these) and give it the address for where to look for the position vertices. Then we point the compiler to the position in the vertex shader to load this data. We do the same for the normal vertices and texture coordinates, though remember that texture coordinates use `vec2`s.  For reference, our vertex shader contains the following:
```cpp
layout (location = 0) in vec3 aPos; // position vertex
layout (location = 1) in vec3 aNormal; // normal vertex
layout (location = 2) in vec2 aTexCoords; // texture vertex
layout (location = 3) in mat4 aInstance; // instance matrix
```

The reason we do this is because we access each of these locations in our vertex shader. In it, we have the `POSITION_VBO`, `NORMAL_VBO`, and `TEXTURE_VBO` set to constant values which we predefined in our header file. The element buffer object, which stores indices, is not in our header file, and so we don't use `glVertexAttribPointer` with it.

The final chunk here starts using the instance buffer. Because we're going to be instancing our models, we need a new model matrix for each instance. As a result, we will store a `mat4` at this address. The model matrix stores the transform of each new instance which we can transform in the CPU. However, GLSL can only store up to 4 floats in a single item. In other words, a `vec4`. This means that to store a $4\times4$ matrix, we will need 4 `vec4` objects. Hence, we tell the compiler to load a `mat4` using a *stride* of a `vec4`, 4 times. This will store one `vec4` 4 times, creating our matrix. The final line, `glVertexAttribDivisor(INSTANCE_VBO + i, 1)`, tells the compiler that we wish to start instancing and to use 1 copy of all vertices, indices, and textures per instance. ^mat-4-expl

##### render
```cpp
void Mesh::render(unsigned int nInstances, const mat4* model_matrix) {
	glBindVertexArray(VAO);
	glBindBuffer(GL_ARRAY_BUFFER, IBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(mat4) * nInstances, &model_matrix[0], GL_DYNAMIC_DRAW);
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		unsigned int mIndex = m_Meshes[i].materialIndex;
		assert(mIndex < m_Textures.size());

		if (m_Textures[mIndex]) m_Textures[mIndex]->bind(GL_TEXTURE0);
		glDrawElementsInstancedBaseVertex(
			GL_TRIANGLES,
			m_Meshes[i].n_Indices,
			GL_UNSIGNED_INT,
			(void*)(sizeof(unsigned int) * m_Meshes[i].baseIndex),
			nInstances,
			m_Meshes[i].baseVertex
		);
	}
	glBindVertexArray(0); // prevent VAO from being changed externally
}
```

This is the function that actually renders our mesh. We start by binding the VAO to start drawing. We then tell the compiler to start drawing instances using the transformation matrices we provided. `nInstances` is the amount of instances we want, which can be any non-zero integer, and `const mat4* model_matrix` is an array of matrices we use to transform our instances.

For each mesh, we ensure that we have loaded every texture before rendering. We bind the current mesh's texture as the current one if we have and continue.

`glDrawElementsInstancedBaseVertex` is a few things:
- "elements" means we're using indexed vertices (indices)
- "instanced" means we're rendering a number of instances
- "base vertex" means we're using a new index in our set of vertices for each mesh. This is how we properly use a Structure of Arrays (SoA) format; otherwise we'd have no idea which index to start from for any one mesh.

We use this function to draw our model. We tell it to use `GL_TRIANGLES` because our mesh is triangulated. We give it the number of indices we're working with and tell it the type of the previous count. We then tell it where to find our indices, which starts from the base index of the current mesh. Then we tell it how many instances we want to render. Finally, we specify that we want to move up by `baseVertex` every time we use a new mesh to separate them, again due to us using SoA. We end this function by unbinding our VAO to avoid accidentally modifying our render.

Now, we can render any model we want as many times as we want. But first, we need to actually pass these values into our vertex shader. This is addressed in the next section.

#### Full Code
##### Mesh.h
```cpp
#pragma once
#include <filesystem>
#include <string>
#include <map>
#include <stdio.h>
#include <stddef.h>
#include <math.h>
#include <vector> // STL dynamic memory.
#include <assert.h> // STL dynamic memory.

#include <assimp/cimport.h> // scene importer
#include <assimp/Importer.hpp>
#include <assimp/scene.h> // collects data
#include <assimp/postprocess.h> // various extra operations

#include <GL/glew.h>
#include <GL/freeglut.h>

#include "Help.h"
#include "Texture.h"

#define AI_LOAD_FLAGS aiProcess_Triangulate | aiProcess_PreTransformVertices
#define MODELDIR(m) "../../Models/" + m.substr(0, m.find(".")) + "/"

class Mesh
{
public:
	struct MeshObject {
		MeshObject() {
			n_Indices = 0;
			baseVertex = 0;
			baseIndex = 0;
			materialIndex = 0;
		}
		unsigned int n_Indices;
		unsigned int baseVertex;
		unsigned int baseIndex;
		unsigned int materialIndex;
	};

	Mesh() { ; }
	Mesh(std::string mesh_name) { loadMesh(mesh_name); }
	bool loadMesh(std::string mesh_name);
	bool initScene(const aiScene*, std::string);
	void initSingleMesh(const aiMesh*);
	bool initMaterials(const aiScene*, std::string);
	void loadColours(const aiMaterial*, int);
	void populateBuffers();
	void render(unsigned int, const mat4*);

#define POSITION_VBO 0
#define NORMAL_VBO 1
#define TEXTURE_VBO 2
#define INSTANCE_VBO 3

	mat4 mat;
	unsigned int VAO; // mesh vao
	unsigned int p_VBO; // position vbo
	unsigned int n_VBO; // normal vbo
	unsigned int t_VBO; // texture vbo
	unsigned int EBO; // index (element) vbo (ebo)
	unsigned int Instance; // instance vbo (ibo)

	std::vector<vec3> m_Positions;
	std::vector<vec3> m_Normals;
	std::vector<vec2> m_TexCoords;
	std::vector<unsigned int> m_Indices;
	std::vector<MeshObject> m_Meshes;
	std::vector<Texture*> m_Textures;
};
```

##### Mesh.cpp
```cpp
#include "Mesh.h"

bool Mesh::loadMesh(std::string file_name) {
	glGenVertexArrays(1, &VAO);
	glBindVertexArray(VAO);
	glGenBuffers(1, &p_VBO);
	glGenBuffers(1, &n_VBO);
	glGenBuffers(1, &t_VBO);
	glGenBuffers(1, &EBO);
	glGenBuffers(1, &Instance);
	
	std::string rpath = MODELDIR(file_name) + file_name;
	const aiScene* scene = aiImportFile(
		rpath.c_str(),
		AI_LOAD_FLAGS
	);

	bool valid_scene = false;

	if (!scene) {
		fprintf(stderr, "ERROR: reading mesh %s\n%s", rpath.c_str(), aiGetErrorString());
		valid_scene = false;
	}
	else {
		valid_scene = initScene(scene, file_name);
	}

	glBindVertexArray(0); // avoid modifying VAO between loads
	return valid_scene;
}

bool Mesh::initScene(const aiScene* scene, std::string file_name) {
	m_Meshes.resize(scene->mNumMeshes);
	m_Textures.resize(scene->mNumMaterials);
	//m_Materials.resize(scene->mNumMaterials);

	unsigned int nvertices = 0;
	unsigned int nindices = 0;

	// Count all vertices and indices
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		m_Meshes[i].materialIndex = scene->mMeshes[i]->mMaterialIndex; // get current material index
		m_Meshes[i].n_Indices = scene->mMeshes[i]->mNumFaces * 3; // there are 3 times as many indices as there are faces (since they're all triangles)
		m_Meshes[i].baseVertex = nvertices; // index of first vertex in the current mesh
		m_Meshes[i].baseIndex = nindices; // track number of indices

		// Move forward by the corresponding number of vertices/indices to find the base of the next vertex/index
		nvertices += scene->mMeshes[i]->mNumVertices;
		nindices += m_Meshes[i].n_Indices;
	}

	// Reallocate space for structure of arrays (SOA) values
	m_Positions.reserve(nvertices);
	m_Normals.reserve(nvertices);
	m_TexCoords.reserve(nvertices);
	m_Indices.reserve(nindices);

	// Initialise meshes
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		const aiMesh* am = scene->mMeshes[i];
		initSingleMesh(am);
	}

	if (!initMaterials(scene, file_name)) {
		return false;
	}

	populateBuffers();
	return glGetError() == GL_NO_ERROR;
}

void Mesh::initSingleMesh(const aiMesh* amesh) {
	const aiVector3D Zero3D(0.0f, 0.0f, 0.0f);

	// Populate the vertex attribute vectors
	for (unsigned int i = 0; i < amesh->mNumVertices; i++) {
		const aiVector3D& pPos = amesh->mVertices[i];
		const aiVector3D& pNormal = amesh->mNormals[i];
		const aiVector3D& pTexCoord = amesh->HasTextureCoords(0) ? amesh->mTextureCoords[0][i] : Zero3D;

		m_Positions.push_back(vec3(pPos.x, pPos.y, pPos.z));
		m_Normals.push_back(vec3(pNormal.x, pNormal.y, pNormal.z));
		m_TexCoords.push_back(vec2(pTexCoord.x, pTexCoord.y));
	}

	// Populate the index buffer
	for (unsigned int i = 0; i < amesh->mNumFaces; i++) {
		const aiFace& Face = amesh->mFaces[i];
		assert(Face.mNumIndices == 3);
		m_Indices.push_back(Face.mIndices[0]);
		m_Indices.push_back(Face.mIndices[1]);
		m_Indices.push_back(Face.mIndices[2]);
	}
}

bool Mesh::initMaterials(const aiScene* scene, std::string file_name) {
	bool valid = true;
	std::string dir = MODELDIR(file_name);
	for (unsigned int i = 0; i < scene->mNumMaterials; i++) {
		const aiMaterial* pMaterial = scene->mMaterials[i];
		m_Textures[i] = NULL;

		if (pMaterial->GetTextureCount(aiTextureType_DIFFUSE) > 0) {
			aiString Path;

			if (pMaterial->GetTexture(aiTextureType_DIFFUSE, 0, &Path, NULL, NULL, NULL, NULL, NULL) == AI_SUCCESS) {
				std::string p(Path.data);

				if (p.substr(0, 2) == ".\\") {
					p = p.substr(2, p.size() - 2);
				}

				std::string fullPath = dir + p;
				m_Textures[i] = new Texture(fullPath, GL_TEXTURE_2D);

				if (!m_Textures[i]->load()) {
					printf("Error loading texture '%s'\n", fullPath.c_str());
					delete m_Textures[i];
					m_Textures[i] = NULL;
					valid = false;
				}
				else {
					printf("Loaded texture '%s'\n", fullPath.c_str());
				}
			}
		}
	}
	return valid;
}

void Mesh::populateBuffers() {
	glBindBuffer(GL_ARRAY_BUFFER, p_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_Positions[0]) * m_Positions.size(), &m_Positions[0], GL_STATIC_DRAW);
	glVertexAttribPointer(POSITION_VBO, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(POSITION_VBO);

	glBindBuffer(GL_ARRAY_BUFFER, n_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_Normals[0]) * m_Normals.size(), &m_Normals[0], GL_STATIC_DRAW);
	glVertexAttribPointer(NORMAL_VBO, 3, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(NORMAL_VBO);

	glBindBuffer(GL_ARRAY_BUFFER, t_VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(m_TexCoords[0]) * m_TexCoords.size(), &m_TexCoords[0], GL_STATIC_DRAW);
	glVertexAttribPointer(TEXTURE_VBO, 2, GL_FLOAT, GL_FALSE, 0, 0);
	glEnableVertexAttribArray(TEXTURE_VBO);

	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(m_Indices[0]) * m_Indices.size(), &m_Indices[0], GL_STATIC_DRAW);

	glBindBuffer(GL_ARRAY_BUFFER, Instance);
	for (unsigned int i = 0; i < 4; i++) {
		glEnableVertexAttribArray(INSTANCE_VBO + i);
		glVertexAttribPointer(INSTANCE_VBO + i, 4, GL_FLOAT, GL_FALSE, sizeof(mat4), (const void*)(i * sizeof(vec4)));
		glVertexAttribDivisor(INSTANCE_VBO + i, 1); // tell OpenGL this is an instanced vertex attribute.
	}
}

void Mesh::render(unsigned int nInstances, const mat4* model_matrix) {
	glBindVertexArray(VAO);
	glBindBuffer(GL_ARRAY_BUFFER, Instance);
	glBufferData(GL_ARRAY_BUFFER, sizeof(mat4) * nInstances, &model_matrix[0], GL_DYNAMIC_DRAW);
	for (unsigned int i = 0; i < m_Meshes.size(); i++) {
		unsigned int mIndex = m_Meshes[i].materialIndex;
		assert(mIndex < m_Textures.size());

		if (m_Textures[mIndex]) m_Textures[mIndex]->bind(GL_TEXTURE0);
		glDrawElementsInstancedBaseVertex(
			GL_TRIANGLES,
			m_Meshes[i].n_Indices,
			GL_UNSIGNED_INT,
			(void*)(sizeof(unsigned int) * m_Meshes[i].baseIndex),
			nInstances,
			m_Meshes[i].baseVertex
		);
	}
	glBindVertexArray(0); // prevent VAO from being changed externally
}
```
### 1.3: Shaders
Before anything is actually drawn on screen, we need to tell the GPU what to draw. We do that in our vertex and fragment shaders.

#### Shader Class
Our shader class again takes heavy inspiration from LearnOpenGL's class. You may have `AddShader` and `CompileShader` functions already; these can be moved to this class for simplicity.

##### Shader.h
```cpp
#pragma once
#include <GLM/vec3.hpp>
#include <GLM/vec2.hpp>
#include <GL/glew.h>
#include <GL/freeglut.h>
#include <vector>
#include <string>
#include <iostream>
#include "Help.h"

class Shader
{
public:
	GLuint ID = 0;
	std::string name;
	Shader() {}
	Shader(std::string shader_name, const char* vertex_shader_path, const char* fragment_shader_path) {
		name = shader_name;
		ID = CompileShaders(vertex_shader_path, fragment_shader_path);
	}

	void AddShader(GLuint ShaderProgram, const char* pShaderText, GLenum ShaderType);
	GLuint CompileShaders(const char* pVS, const char* pFS);


	// activate the shader
	// ------------------------------------------------------------------------
	void use()
	{
		glUseProgram(ID);
	}
	// utility uniform functions
	// ------------------------------------------------------------------------
	void setBool(const std::string& name, bool value) const
	{
		glUniform1i(glGetUniformLocation(ID, name.c_str()), (int)value);
	}
	void setInt(const std::string& name, int value) const
	{
		glUniform1i(glGetUniformLocation(ID, name.c_str()), value);
	}
	void setFloat(const std::string& name, float value) const
	{
		glUniform1f(glGetUniformLocation(ID, name.c_str()), value);
	}
	void setVec2(const std::string& name, vec2 value) const
	{
		glUniform2f(glGetUniformLocation(ID, name.c_str()), value.v[0], value.v[1]);
	}
	void setVec2(const std::string& name, float x, float y) const
	{
		glUniform2f(glGetUniformLocation(ID, name.c_str()), x, y);
	}
	void setVec3(const std::string& name, vec3 value) const
	{
		glUniform3f(glGetUniformLocation(ID, name.c_str()), value.v[0], value.v[1], value.v[2]);
	}
	void setVec3(const std::string& name, float x, float y, float z) const
	{
		glUniform3f(glGetUniformLocation(ID, name.c_str()), x, y, z);
	}
	void setMat4(const std::string& name, const mat4 &mat) const
	{
		glUniformMatrix4fv(glGetUniformLocation(ID, name.c_str()), 1, GL_FALSE, mat.m);
	}
};
```

##### Shader.cpp
```cpp
#include "Shader.h"

void Shader::AddShader(GLuint ShaderProgram, const char* pShaderText, GLenum ShaderType)
{
	// Create a shader object
	GLuint ShaderObj = glCreateShader(ShaderType);

	if (ShaderObj == 0)
	{
		fprintf(stderr, "Error creating shader type %d\n", ShaderType);
		exit(0);
	}
	// Bind the source code to the shader, this happens before compilation
	glShaderSource(ShaderObj, 1, (const GLchar**)&pShaderText, NULL);
	// Compile the shader and check for errors
	glCompileShader(ShaderObj);
	GLint success;
	// Check for shader related errors using glGetShaderiv
	glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);
	if (!success)
	{
		GLchar InfoLog[1024];
		glGetShaderInfoLog(ShaderObj, 1024, NULL, InfoLog);
		fprintf(stderr, "Error compiling shader type %d: '%s'\n", ShaderType, InfoLog);
		exit(1);
	}
	// Attach the compiled shader object to the program object
	glAttachShader(ShaderProgram, ShaderObj);
}

GLuint Shader::CompileShaders(const char* pVS, const char* pFS)
{
	//Start the process of setting up our shaders by creating a program ID
	//Note: we will link all the shaders together into this ID
	GLuint shaderProgramID = glCreateProgram();
	if (shaderProgramID == 0) {
		std::cerr << "Error creating shader program..." << std::endl;
		std::cerr << "Press enter/return to exit..." << std::endl;
		std::cin.get();
		exit(1);
	}

	// Create two shader objects, one for the vertex, and one for the fragment shader
	AddShader(shaderProgramID, Help::readFile(pVS).c_str(), GL_VERTEX_SHADER);
	AddShader(shaderProgramID, Help::readFile(pFS).c_str(), GL_FRAGMENT_SHADER);

	GLint Success = 0;
	GLchar ErrorLog[1024] = { '\0' };
	// After compiling all shader objects and attaching them to the program, we can finally link it
	glLinkProgram(shaderProgramID);
	// check for program related errors using glGetProgramiv
	glGetProgramiv(shaderProgramID, GL_LINK_STATUS, &Success);
	if (Success == 0) {
		glGetProgramInfoLog(shaderProgramID, sizeof(ErrorLog), NULL, ErrorLog);
		std::cerr << "Error linking shader program: " << ErrorLog << std::endl;
		std::cerr << "Press enter/return to exit..." << std::endl;
		std::cin.get();
		exit(1);
	}

	// program has been successfully linked but needs to be validated to check whether the program can execute given the current pipeline state
	glValidateProgram(shaderProgramID);
	// check for program related errors using glGetProgramiv
	glGetProgramiv(shaderProgramID, GL_VALIDATE_STATUS, &Success);
	if (!Success) {
		glGetProgramInfoLog(shaderProgramID, sizeof(ErrorLog), NULL, ErrorLog);
		std::cerr << "Invalid shader program: " << ErrorLog << std::endl;
		std::cerr << "Press enter/return to exit..." << std::endl;
		std::cin.get();
		exit(1);
	}
	
	// Note: this program will stay in effect for all draw calls until you replace it with another or explicitly disable its use
	return shaderProgramID;
}
```
#### Vertex Shader
In case you forgot or didn't know, the vertex shader is a program the GPU uses when drawing vertices. We want to draw each vertex of the mesh we use in view space, as well as the texture we give it.

We start by declaring the position in memory of the data we want to pass in:
```cpp
#version 330
layout (location = 0) in vec3 aPos; // position vertex
layout (location = 1) in vec3 aNormal; // normal vertex
layout (location = 2) in vec2 aTexCoords; // texture vertex
layout (location = 3) in mat4 aInstance; // instance matrix
```

(We're using OpenGL v3.3.) The vertices of our mesh are in 3D space, as well as our normals, so they are `vec3`. Our texture is 2D, so any coordinates on it will also be 2D, hence `vec2`. Finally, the transform of our model is a `mat4`. You can check the [[#^mat-4-expl|Mesh section]] for a full explanation, the the basic of it is that we're storing the transform (position, rotation, and scale) of each instance in the this matrix. Before, you may have had `uniform mat4 model` in your vertex shader, but since we're no longer giving the shader a transform directly, this isn't necessary.

We want to send the position, normals, and texture coordinates of our vertex shader to the fragment shader so it knows what to render and different distances and rotations, so we output this data:
```cpp
out vec3 FragPos;
out vec3 Normal;
out vec2 TexCoords;
```

And we still give our instance the view and projection matrices of the world, though we'd only be passing it into our shader once.
```cpp
uniform mat4 view;
uniform mat4 projection;
```

Finally, we have our `main` function. This function is very simple; all we do is pass our vertex data into our outputs, with small amounts of transforming.
```cpp
void main() {
    FragPos = vec3(aInstance * vec4(aPos, 1.0));
    Normal = mat3(transpose(inverse(aInstance))) * aNormal;
    TexCoords = aTexCoords;
    gl_Position = projection * view * vec4(FragPos, 1.0);
}
```

We get the position of each fragment (`FragPos`) by multiplying our instance matrix with our position vertex. We calculate the transpose of the inverse of our instance model and multiply it by our normal vertex to get our output normal. I have no idea why we have to do this, but you can find potential reasons [here](https://stackoverflow.com/questions/13654401/why-transform-normals-with-the-transpose-of-the-inverse-of-the-modelview-matrix), [here](https://www.lighthouse3d.com/tutorials/glsl-12-tutorial/the-normal-matrix/), and [here](https://paroj.github.io/gltut/Illumination/Tut09%20Normal%20Transformation.html). We simply give our output texture coordinates the ones we get in, and finally get the position of this vertex by multiplying our projection, view, and fragment position matrices.

This code is sufficient to render each of our instances using the GPU instead of in normal code. Lighting is handled by the fragment shader.

#### Fragment Shader (Temporary)
I will go over lighting in a future section. For now, you can have the following piece of code to use with your current vertex shader:
```cpp
#version 330
out vec4 FragColor;
  
struct Material {
    sampler2D diffuse;
    sampler2D specular;    
    float shininess;
};
  
struct Light {
    vec3 position;
  
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
  
in vec3 FragPos;
in vec3 Normal;  
in vec2 TexCoords;
uniform vec3 viewPos;
uniform Material material;
uniform Light light;
  
void main()
{
    // ambient
    vec3 ambient = light.ambient * texture(material.diffuse, TexCoords).rgb;
    // diffuse
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(light.position - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = light.diffuse * diff * texture(material.diffuse, TexCoords).rgb;  
    // specular
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);  
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = light.specular * spec * texture(material.specular, TexCoords).rgb;  
    vec3 result = ambient + diffuse + specular;
    FragColor = vec4(result, 1.0);
}
```

This fragment shader will use a simple form of the Phong Illumination Model to light all vertices the same colour. It has been sourced from here: https://learnopengl.com/code_viewer_gh.php?code=src/2.lighting/4.2.lighting_maps_specular_map/4.2.lighting_maps.fs. 


### 1.4: Blender Tutorial
This section is a simple blender tutorial you can follow to get comfortable exporting simple models with textures to .obj files. It is also designed to get you more familiar with whatever file system you chose to organise your files with.

You can check models for free with:
- 3D Viewer, an online open-source webpage that lets you load and view models
	- https://3dviewer.net/
- Microsoft 3D Viewer, Microsoft's free object loader available on the Microsoft Store
	- https://apps.microsoft.com/detail/9NBLGGH42THS?ocid=pdpshare&hl=en-us&gl=US

#### 1.4.1: Downloading Blender
You can download Blender here:

https://www.blender.org/download/

I am using Blender 3.3. The newest version is v4+, but I don't think there will many changes for GUI or settings between version. Regardless, if you want to follow along using the exact version of Blender I used, you can download Blender v3.3 LTS here: https://www.blender.org/download/lts/3-3/

#### 1.4.2: First Cube
When you open Blender, you should see a screen like this:
![[Pasted image 20231205152451.png|500]]

Click **General** to continue. You will now see a cube in front of you.
![[Pasted image 20231205152524.png|500]]

Take note of the window and get familiar with it. You can use the middle mouse button to rotate the scene or the grid of coloured icons to choose a direction. 

#### 1.4.3: Materials (Textures)
Click the little red/block ball "Material Properties". 
![[Pasted image 20231205152930.png|500]]

Click the yellow circle to the left of "Base Color" and select "Image Texture" from the menu that opens. By this point, you should have your hierarchy sorted out (see [[#Organisation and Hierarchy]] for how I modelled mine). For example, let's say I want to create a new model called "wood_plank". I would first create that folder in my Models folder, then create a Textures folder within that one and save any images I wish to use there. You can (and should) save this Blender file to that folder (i.e., the wood_plank folder). Make sure you have the images you wish to use in your Textures folder ***before*** you export this file.

Select the image you wish to use from your Textures folder. Nothing will happen immediately, because you are in Solid mode. You can enable Render or Material mode in the top right of the model window and selecting either one of the two circles to the right.
![[Pasted image 20231205154109.png|500]]

You have now successfully applied a texture to a model.
![[Pasted image 20231205154306.png|500]]

#### 1.4.4 Multiple Textures
You may wish to load more than one texture on your model. To add another texture, select the `+` next to the list of material slots and press "New" when it comes up.
![[Pasted image 20231205154501.png|500]]
![[Pasted image 20231205154519.png|500]]

Follow the same steps as in [[#1.4.3 Materials (Textures)]] to load in a new texture. You can also rename materials by selecting the name in the text field just below the list of materials.

To actually load in a new texture, we need to have a face to put it. First, go to the top left of the model window and click the dropdown "Object Mode" and select "Edit Mode". 
![[Pasted image 20231205154751.png|500]]

Edit Mode allows us to actually change the way parts of the model are rendered. Click "Face Mode" of the three new buttons that appear next to the dropdown.
![[Pasted image 20231205154906.png|500]]
 Let's say we want to load in our second texture on the top face. To do that, click away from the model (into empty space) and then click the top face. You'll also notice the material window has changed, giving us the "Assign", "Select", and "Deselect" options. Pick the second texture from your list and then press "Assign".
 ![[Pasted image 20231205155112.png|500]]
 
Your second texture has been loaded on that face. Play around with this and select other faces to change their textures, or load in more textures to use.

#### 1.4.5 UV Editing
You may notice that the texture is too big on the model. The texture I loaded on the top of my model looks like this (obtained from LearnOpenGL @ https://learnopengl.com/img/textures/container2.png):
![[container2.png|300]]

but only the middle portion is actually rendered on the face. To fix this, we'll have to change how the texture is rendered on the model.

Go the the top of the window and select "UV Editing" (not "UV"; that's different, though useful for later). You'll get a second window looking like the following:
![[Pasted image 20231205155531.png|500]]

You can change the texture you wish to edit from the dropdown at the top. You'll also see that your model is no longer in Render/Material mode. You can go back and re-enable either mode, though you may need to scroll to the right on the right-hand screen to find it.
![[Pasted image 20231205155651.png|500]]

While still in Edit Mode and Face Mode, select the top-most face of the model and the texture it uses. The area of the texture the face uses is incorrect in our case, because we wish to use the entire texture on the face. In either window, press the 'U' key and select "Cube Projection". Alternatively, select "UV" from the top of the right-hand window (not "UV Editing", we already have that open).
![[Pasted image 20231205155949.png|500]]
![[Pasted image 20231205160041.png|500]]

Now we have mapped the entire texture the the face we selected. We can do this to any face we like given the texture we want.
![[Pasted image 20231205160244.png|500]]

Now, let's say we want to change one of the faces. Perhaps by rotating one of them 90 degrees. We can actually do that in the UV editor! Select the face you wish to rotate in the model window, and then, in the UV editor, select the Rotate button (or press 'R'). Select the vertices you wish to rotate. In our case we want to rotate everything, so we left-click and drag over all the corners of the texture (or press 'A'). Now, drag with your mouse to rotate the texture.
![[Pasted image 20231205160545.png|500]]

You can hold down CTRL to snap to a more consistent angle. Left click to disable rotation.

![[Pasted image 20231205161024.png|500]]

Our texture has been rotated 90 degrees, just the way we like it.

#### 1.4.6: Export
We will now export our model the using .obj format.

Save your project. Then, in the top left corner, go to `File -> Export -> Wavefront (.obj)`
![[Pasted image 20231205161400.png|500]]

In the resulting window, make sure you've navigated to the folder you wish to store the model in (if you saved your Blender file there, you should be here already). Make sure to tick "Triangulated Mesh" in the export menu, as we will want to use triangulated meshes with OpenGL. 
![[Pasted image 20231205161617.png|500]]

With that, you have successfully created a new simple textured model in Blender.

#### 1.4.7: Embedded Textures
Blender actually lets you draw and create materials within the editor and export them with your models. These are known as "embedded materials". They can be saved externally to be edited with an image editor or drawing app, or designed right in Blender. Adding them to a model is simple enough, but a few changes will need to be made.

First, create a new Blender project. This time, we'll create a cone. Select the cube and press `Delete` or 'X' to delete the cube, then press Shift + 'A' to add a new mesh. [pic] Go to `Mesh -> Cone`. [pic of cone]



## Part 1.5 - Testing
(Not unit testing; sorry if you freaked out). This is a halfway point, a section made to check if you've set up everything correctly. For simplicity's sake, I have pushed all my imports into a separate `main.h` file and only import that into `main.cpp`. You don't have to do this, but I found it easier to manage.

#### Header (Main.h)
We'll start with this list of imports:
```cpp
#include <windows.h>
#include <mmsystem.h>
#include <iostream>
#include <string>
#include <map>
#include <stdio.h>
#include <stddef.h>
#include <math.h>
#include <vector> // STL dynamic memory.

// OpenGL includes
#include <GL/glew.h>
#include <GL/freeglut.h>

// Assimp includes
#include <assimp/cimport.h> // scene importer
#include <assimp/scene.h> // collects data
#include <assimp/postprocess.h> // various extra operations

// Project includes
#include "maths_funcs.h"
#include "mesh.h"
#include "shader.h"
#include "help.h"

#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```

Note what we're still including `stb_image.h` despite the fact we already included it in the Textures section. I don't get it, either.

Remember the code we used to load in textures? Because we only told it to use the name of a texture, we only need to give it that name when we call it. We can store loads of textures in our `main.h` function using `#define` and access them whenever we want.
```cpp
#define MESH_TEST "test.obj"
#define MESH_TESTCUBE "test_cube.obj"
#define MESH_CYLINDER "test_cylinder.obj"
#define MESH_SPIDER "spider.obj"
#define MESH_GROUND "test_ground.obj"
#define MESH_PLANK "wood_plank.obj"
```

#### Implementation (Main.cpp)
We obviously start by including the `main.h` file. I elected to add `using namespace std` at the top since nothing has clashed with it (yet). If not, you'll have to have `std::` in front of anything you want to use from that namespace. We need to setup the following functions:
- `init`
- `display`
- `updateScene`

If you're working from Lab 3 code, you will already have these functions set up in some way. Keyboard and mouse input will be discussed in the Camera section. 

##### init
We start by initialising our scene. We define our vertex and fragment shaders here and use them.
```cpp
Shader* s = new Shader("base", "../../Shaders/vertex.glsl", "../../Shaders/fragment.glsl");
s->use();
shaders[s->name] = s;
```
We also keep a dictionary of shaders so we can call them by a name any time we need to.

Next, we load our meshes. We can start by loading that cube from eariler:
```cpp
Mesh* m = new Mesh();
if (!m->loadMesh(MESH_PLANK)) {
	cout << "\n\nfailed to load mesh :(\n";
}
meshes.push_back(m);
```
If the mesh fails to load, we want to know about it. We also keep our meshes in a list (C++ has `vector`, similar to `ArrayList` in Java) so we can access them when we render them. 

Since we're using instancing, we want to create new positions for each model instance. You can have this code to create 1,000 random positions controlled by a spreading factor `spread`:
```cpp
srand(time(nullptr));
float offset = 1.0f;
for (int z = -10; z < 10; z += 2) {
	for (int y = -10; y < 10; y += 2) {
		for (int x = -10; x < 10; x += 2) {
			vec3 translation;
			translation.v[0] = (float)x + offset * (rand() % spread);
			translation.v[1] = (float)y + offset * (rand() % spread);
			translation.v[2] = (float)z + offset * (rand() % spread);
			translations.push_back(translation);
		}
	}
}
```
where `translations` is a vector of `vec3` objects. Keep your `translations` vector in your .h file so you don't lose these values.

##### display
This function tells OpenGL what to display to the screen. You will probably have something that looks like this already:
```cpp
// tell GL to only draw onto a pixel if the shape is closer to the viewer
glEnable(GL_DEPTH_TEST); // enable depth-testing
glEnable(GL_BLEND); // enable depth-testing
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
glDepthFunc(GL_LESS); // depth-testing interprets a smaller value as "closer"
glClearColor(0.1f, 0.1f, 0.3f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

I won't be explaining what this does, so make sure to visit https://www.docs.gl/ to learn more.

We now want to use our shader, which we can do with:
```cpp
shaders["base"]->use();
```

We called this shader "base" when creating it.
Next, we give it information about the materials we want the fragment shader to use:
```cpp
shaders["base"]->setVec3("viewPos", 0.0f, 0.0f, -10.0f);
shaders["base"]->setFloat("material.shininess", 64.0f);
shaders["base"]->setInt("material.diffuse", 0);
shaders["base"]->setInt("material.specular", 1);
```

Then information about the light source:
```cpp
shaders["base"]->setVec3("light.position", 0.0f, 0.0f, -10.0f);
shaders["base"]->setVec3("light.ambient", vec3(0.5f)); // vec3(0.5f, 0.5f, 0.5f)
shaders["base"]->setVec3("light.diffuse", vec3(0.8f)); // ditto
shaders["base"]->setVec3("light.specular", vec3(0.5f)); // ditto
```

Next, we want to define our view and projection matrix for the scene. Again, you likely already have this code, but in case you don't:
```cpp
mat4 view = identity_mat4();
view = translate(view, vec3(0.6f, 0.3f, 0.8f));
mat4 persp_proj = perspective(45.0f, (float)width / (float)height, 0.1f, 1000.0f);
shaders["base"]->setMat4("view", view);
shaders["base"]->setMat4("projection", persp_proj);
```
The `45.0f` in the `perspective` function is our field-of-view (FOV). `width` and `height` can be defined in your `main.h` file as whatever you like, such as 800 and 600, respectively. Then, we give these matrices to our shader for rendering.

Now, we render each instance. We want to render 1,000 instances, so we can use the following code:
```cpp
const unsigned int numInstances = 1000;
mat4 models[numInstances];
for (int i = 0; i < numInstances; i++) {
	models[i] = translate(identity_mat4(), translations[i]);
}
meshes[0]->render(numInstances, models);
```
This code will create an array of `mat4` of length 1,000 and translate each matrix with a corresponding translation from the list we created in the `init` function. Finally, we render every instance with a single render call using the number of instances we have and the transforms of each instance. We end the function with:
```cpp
glutSwapBuffers();
```
to refresh the screen on every call.

##### updateScene
This function is called once per frame. In it, we would like to control things such as the passage of time and cleaning whatever we output on each frame. The function is very small, so I'll give it to you here:
```cpp
void updateScene() {
	static DWORD last_time = 0;
	DWORD curr_time = timeGetTime();
	if (last_time == 0)
		last_time = curr_time;
	Help::delta = (curr_time - last_time) * 0.001f;
	last_time = curr_time;

	camera.processMovement();
	p = fmodf(p + (Help::delta * 5.0f), 360.0f);
	tp += p;
	// Draw the next frame
	glutPostRedisplay();
}
```

##### main
In the `main` function, you are best off leaving everything untouched for now. It should look roughly like this:
```cpp
int main(int argc, char** argv) {

	// Set up the window
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
	glutInitWindowSize(width, height);
	glutCreateWindow("My Project");

	// Tell glut where the display function is
	glutDisplayFunc(display);
	glutIdleFunc(updateScene);
	// glutKeyboardFunc(keypress); // leaving this out for now

	// A call to glewInit() must be done after glut is initialized!
	GLenum res = glewInit();
	// Check for any errors
	if (res != GLEW_OK) {
		fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
		return 1;
	}
	// Set up your objects and shaders
	init();
	// Begin infinite event loop
	glutMainLoop();
	return 0;
}
```

If all goes well, you should get something similar to this:
![[Pasted image 20231205172258.png|700]]

And if not, feel free to tweak the view matrix, `viewPos` shader value, or spread until you get something you like. If nothing is appearing, try changing the spread to 1 and checking that anything shows up.
#### Full Code
##### main.h
```cpp
#pragma once

#define NOMINMAX
#include <limits>

// Windows includes (For Time, IO, etc.)
#include <windows.h>
#include <mmsystem.h>
#include <iostream>
#include <string>
#include <map>
#include <stdio.h>
#include <stddef.h>
#include <math.h>
#include <vector> // STL dynamic memory.

// OpenGL includes
#include <GL/glew.h>
#include <GL/freeglut.h>

// Assimp includes
#include <assimp/cimport.h> // scene importer
#include <assimp/scene.h> // collects data
#include <assimp/postprocess.h> // various extra operations

// Project includes
#include "maths_funcs.h"
#include "mesh.h"
#include "shader.h"
#include "help.h"

#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"


#pragma region MESH_NAMES
// Current working meshes
#define MESH_TEST "test.obj"
#define MESH_TESTCUBE "test_cube.obj"
#define MESH_CYLINDER "test_cylinder.obj"
#define MESH_SPIDER "spider.obj"
#define MESH_GROUND "test_ground.obj"
#define MESH_PLANK "wood_plank.obj"
#pragma endregion MESH_NAMES

std::map<std::string, Shader*> shaders;
std::vector<Mesh*> meshes;
std::vector<vec3> translations;
int spread = 100;
```

##### main.cpp
```cpp
#include "main.h"
using namespace std;

void display() {
	// tell GL to only draw onto a pixel if the shape is closer to the viewer
	glEnable(GL_DEPTH_TEST); // enable depth-testing
	glEnable(GL_BLEND); // enable depth-testing
	glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
	glDepthFunc(GL_LESS); // depth-testing interprets a smaller value as "closer"
	glClearColor(0.1f, 0.1f, 0.3f, 1.0f);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	// Root of the Hierarchy
	shaders["base"]->use();

	shaders["base"]->setVec3("viewPos", vec3(0.6f, 0.3f, 0.8f));
	shaders["base"]->setFloat("material.shininess", 64.0f);
	shaders["base"]->setInt("material.diffuse", 0);
	shaders["base"]->setInt("material.specular", 1);

	shaders["base"]->setVec3("light.position", 0.0f, 0.0f, -10.0f);
	shaders["base"]->setVec3("light.ambient", vec3(0.5f)); // vec3(0.5f, 0.5f, 0.5f)
	shaders["base"]->setVec3("light.diffuse", vec3(0.8f)); // ditto
	shaders["base"]->setVec3("light.specular", vec3(0.5f)); // ditto

	mat4 view = identity_mat4();
	view = translate(view, vec3(0.6f, 0.3f, 0.8f));
	mat4 persp_proj = perspective(45.0f, (float)width / (float)height, 0.1f, 1000.0f);
	shaders["base"]->setMat4("view", view);
	shaders["base"]->setMat4("projection", persp_proj);

	// Transform each instance
	const unsigned int numInstances = 1000;
	mat4 models[numInstances];
	for (int i = 0; i < numInstances; i++) {
		models[i] = translate(identity_mat4(), translations[i]);
	}
	meshes[0]->render(numInstances, models);
	glutSwapBuffers();
}


void updateScene() {
	static DWORD last_time = 0;
	DWORD curr_time = timeGetTime();
	if (last_time == 0)
		last_time = curr_time;
	Help::delta = (curr_time - last_time) * 0.001f;
	last_time = curr_time;
	
	// Draw the next frame
	glutPostRedisplay();
}

void updateScene() {
	static DWORD last_time = 0;
	DWORD curr_time = timeGetTime();
	if (last_time == 0)
		last_time = curr_time;
	Help::delta = (curr_time - last_time) * 0.001f;
	last_time = curr_time;

	camera.processMovement();
	p = fmodf(p + (Help::delta * 5.0f), 360.0f);
	tp += p;
	// Draw the next frame
	glutPostRedisplay();
}


void init()
{
	srand(time(nullptr));
	float offset = 1.0f;
	for (int z = -10; z < 10; z += 2) {
		for (int y = -10; y < 10; y += 2) {
			for (int x = -10; x < 10; x += 2) {
				vec3 translation;
				translation.v[0] = (float)x + offset * (rand() % spread);
				translation.v[1] = (float)y + offset * (rand() % spread);
				translation.v[2] = (float)z + offset * (rand() % spread);
				translations.push_back(translation);
			}
		}
	}

	Shader* s = new Shader("base", "../../Shaders/tvs.glsl", "../../Shaders/tfs.glsl");
	s->use();
	shaders[s->name] = s;

	// Load meshes to be used
	Mesh* m = new Mesh();
	if (!m->loadMesh(MESH_PLANK)) {
		cout << "\n\nfailed to load mesh :(\n";
	}
	meshes.push_back(m);
}


int main(int argc, char** argv) {

	// Set up the window
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
	glutInitWindowSize(width, height);
	glutCreateWindow("My Project");

	// Tell glut where the display function is
	glutDisplayFunc(display);
	glutIdleFunc(updateScene);
	// glutKeyboardFunc(keypress); // leaving this out for now

	// A call to glewInit() must be done after glut is initialized!
	GLenum res = glewInit();
	// Check for any errors
	if (res != GLEW_OK) {
		fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
		return 1;
	}
	// Set up your objects and shaders
	init();
	// Begin infinite event loop
	glutMainLoop();
	return 0;
}
```
***NOTE:*** This code has not been tested in form. Some things may not work correctly. If something goes wrong, feel free to query me about it.

## Part 2 - Camera
The camera is an integral part of any 3D traversal system. You need to be able to see where you're going, after all. 

We will create a Camera class to hold and define everything we need for 3D movement and looking around the viewport. This class is based off of the one created by LearnOpenGL, which you can find here: https://learnopengl.com/Getting-started/Camera

### 2.1: Theory
#### 2.1.1: Delta Time
The basic theory behind camera movement is that we wish to be able to control our position and viewing angle within 3D space. These can be controlled with the keyboard and mouse. We wish to move around with a certain speed. We can call this our "movement speed". We also with to look around the viewport with a certain speed relative to the speed at which we move our mouse. We can call this our "sensitivity". These two things combined control our rate of movement and can and should be tweaked to make for the most comfortable playing experience. Your movement will update on each frame; that is, on every new draw call, your variables, such as position or viewing angle, will change by the speed you picked.

However, there is a minor problem with this. Those of you familiar with some older games (can't think of any, born in 2003) may have noticed slight differences on movement speed on your console or device compared to others'. Your device updates (refreshes) its screen a variable amount of times a second. Sometimes, this amount can be locked to an upper limit. This is known as the "refresh rate" of your screen and is measured in hertz (hz) or frames per second (FPS). The standard for modern devices is 60hz, though many monitors and even phones offer refresh rates of 90hz or above. 

The problem comes from this variable refresh rate. If your computer runs at 60 FPS, your program will update 60 times a second. That means that your computer will calculate your position from your movement speed 60 times in a single second. If you are at $(0, 5, 9)$ with a speed of $(1, 0, 0)/sec$, you will be at $(60, 5, 9)$ after one second. If your device runs at 90 FPS however, you will be at $(90, 5, 9)$ after one second instead. This is obviously a glaring issue in regards to things like fairness in multiplayer games, as some players may move faster than others simply due to better hardware.

**To combat this, we can make use of something called *delta time*. Delta time is the difference in time between screen updates. You can use it to moderate everything that gets updated a certain number of times per frame by multiplying the change by delta. For example, the time between update calls is obviously much shorter in devices running at 90 FPS compared to those running at 60 FPS. So we multiply our movement speed by this difference in time. If delta for 60 FPS is 1ms, then the delta for 90 FPS will be $\frac{2}{3}$ms. The delta for 30 FPS will then be 2ms. Multiplying these delta values by their refresh rates gives us a normalised 60 frames per second. We can use this idea to get consistent movement speeds for every form of change in our project -- translation, rotation, and scale.** 

In practice, delta is often much smaller than 1 second. Normalising your movement speed will therefore require you to increase its base value to compensate.

#### 2.1.2: Euler Angles - Pitch, Yaw, and Roll
Pitch, yaw, and roll refer to Euler angles representing orientation of an object. In our case, this means the angles of rotation of the viewport. We want to be able to look anywhere we like with the mouse (or keyboard) while also rotating our movement direction, so that we only need to press forward and turn in a certain direction to move in it. 

We can use the right-hand rule to discern what each form of rotation is. Point your right hand to the left, with your index finger pointing left and your thumb pointing up. Point your middle finger towards yourself. Now rotate your hand so that your index finger is pointing up, your thumb to the right, and your middle finger still towards yourself. You have created a graph where:
- your thumb is the positive x-axis
- your index finger is the positive y-axis
- your middle finger is the positive z-axis

![[Pasted image 20231206103744.png|500]]

***Pitch***
Pitch is rotation about the x-axis. If you imagine a disc hung around the x-axis, it would rotate vertically.
![[Pasted image 20231206103951.png|500]]
The pitch determines our *vertical view rotation.* This is how we look up and down.

***Yaw***
Yaw is rotation about the y-axis. Using another disc hung around the y-axis, it would rotate horizontally.
![[Pasted image 20231206104152.png|300]]
The yaw controls our *horizontal view rotation*. This is how we look to the left and right.

***Roll***
Roll is rotation about the z-axis. Using a third disc hung around the z-axis, it would rotate parallel to the viewport.
![[Pasted image 20231206104517.png|500]]
The roll controls our *parallel view rotation*. We won't be using this as there is no real way to control this with our mouse. If you want a comparison as to how it would feel using roll, consider the Nausea effect in Minecraft.

### 2.2: The Camera Class
#### Header (Camera.h)
The header file, like all others, describes the member variables, functions, and imports.
```cpp
#pragma once
#include "maths_funcs.h"
#include "help.h"
#include <GL/freeglut.h>

class Camera
{
public:
	Camera() {
		pos = vec3(0.0f);
		front = vec3(0.0f, 0.0f, -1.0f);
		up = vec3(0, 1, 0);
	}
	void processView(int, int);
	void processMovement();
	mat4 getViewMatrix();

	vec3 pos;
	vec3 front;
	vec3 up;
	vec3 tUP = vec3(0.0f, 1.0f, 0.0f);
	vec3 right;
	float FOV = 60.0f;
	float pitch;
	float yaw;
	float roll;
	float sensitivity = 15.0f;
	float speed = 5.0f;

	// Keyboard movement
	bool FORWARD = false;
	bool BACK = false;
	bool LEFT = false;
	bool RIGHT = false;
	bool UP = false;
	bool DOWN = false;
};
```
`processView` processes camera view movement, and `processMovement` controls camera position movement. `getViewMatrix` gets the `view` matrix we use in our `main.cpp` file, which I'll touch on later.

We have our keyboard controls defined as Booleans for controlling smooth movement. `up` refers to the camera's up vector, whereas `tUP` refers to the true "up" direction in our scene, which is $(0, 1, 0)$. We set our FOV to 60 because I think it's nice, but I also play on "Quake Pro" settings on Minecraft, so you should probably change this if you're not a try-hard. Finally, our sensitivity is how sensitive our rotation is relative to our mouse movement.

#### Implementation (Camera.cpp)
##### processView
`processView` takes in an `x` and `y` parameter which determine how much the mouse has moved since the last update. 

Considering the last 3D game you played that didn't use a cursor directly. Minecraft, for example, disables cursor usage when a menu (inventory, pause) is not open. But where does the cursor go? In this case, the cursor is warped to the centre of the screen (or top left corner in older versions). The reason for this is so that when spinning using your mouse, the mouse cursor doesn't accidentally go off screen and lose focus of the game. 

We use a similar trick here. Using our screen width and height (defined as 800 and 600 wherever you put them), let's put the mid-point of these values in variables.
```cpp
float xPos = Help::width / 2.0;
float yPos = Help::height / 2.0;
```
The mouse cursor will be warped to the middle of the screen, which is what these variables are keeping track of. Next, we'll calculate the pitch and yaw values (we won't calculate the roll since we won't be using it). 

Since we're warping our cursor the the centre of the screen on each update, we need to keep track of our pitch and yaw values, which we keep in our header file. We can get our pitch value by calculating the distance between the middle of the screen and our cursor's x-value before it gets warped, then multiplying by our sensitivity to scale the difference. We do the same with our yaw value.
```cpp
pitch = Help::clamp(pitch + ((y - yPos) * -sensitivity), -89.0f, 89.0f);
yaw = Help::wrap(yaw + ((x - xPos) * sensitivity), 0.0f, 360.0f);
glutWarpPointer(xPos, yPos);
```

We don't allow our pitch to go up to 90 or down to -90 as that would flip our perspective in the view matrix, so we *clamp* it between -89 and +89. We do want to let our yaw go as far as we want, like spinning in circles, so that value is *wrapped* around to 0 if it ever goes beyond 360 and vice versa. Make sure `delta` in included in one of the files you import into your camera class. Finally, we use the GLUT function `glutWarpPointer` to warp our pointer to the middle of the screen.

Next, we want to define the front, right, and up directions of our camera. This is the direction our camera is pointing in and the direction we move in relative to our position. The maths behind it is a little confusing, so you can read LearnOpenGL's explanation of it (or this entire Camera class which I "took inspiration" from): https://learnopengl.com/Getting-started/Camera. Basically, do this:
```cpp
front = normalise(vec3(
	cos(Help::deg2Rad(yaw)) * cos(Help::deg2Rad(pitch)),
	sin(Help::deg2Rad(pitch)),
	sin(Help::deg2Rad(yaw)) * cos(Help::deg2Rad(pitch))
));

right = normalise(cross(front, tUP));
up = normalise(cross(right, front));
```

`Help::deg2Rad` is a helper function I made that converts degree angles to radians, which the sin and cos functions use. You will have a `ONE_DEG_IN_RAD` macro defined somewhere in your `maths_funcs.h` file. If not, you can have it here (for free):
```cpp
// maths_funcs.h
#define ONE_DEG_IN_RAD (2.0f * 3.14159265358979323846) / 360.0f // 0.017444444

// help.cpp
#include maths_funcs.h
float Help::deg2Rad(float val) {
	return val * ONE_DEG_IN_RAD;
}
```

This function is called in `main.cpp`. To get mouse movement, you will need to create a callback function that GLUT calls to interpret mouse movement. You can do this very simply by adding a function:
```cpp
void mouseMoved(int x, int y) {
	camera.processView(x, y);
}
```

This function needs to be known by GLUT, so in our `main` function in the `main.cpp` file, we can add the following line under `glutIdleFunc`:
```cpp
glutPassiveMotionFunc(mouseMoved);
```

This will handle mouse movement where the mouse is not pressed (i.e., not dragged).

Finally, for convenience, add the following line next:
```cpp
glutSetCursor(GLUT_CURSOR_NONE);
```

This line will hide the cursor.

##### getViewMatrix
The view matrix is what actually allows us to change what we see in the viewport. We use the `look_at` function, defined in the `maths_funcs.cpp` file. Alternatively, you can use `glm::lookAt`. For more information on what this function does, you can [read the documentation on it](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/gluLookAt.xml), or look at your notes (Lecture 11 "Viewing", pg 33), or [check this StackOverflow post](https://stackoverflow.com/a/21830455) instead.

All the `getViewMatrix` function does is return the result of the `look_at` function using our camera's position and front and up directions:
```cpp
mat4 Camera::getViewMatrix() {
	return look_at(pos, pos + front, up);
}
```

In our `display` function, change your declaration of the `view` matrix so that you get this:
```cpp
mat4 view = camera.getViewMatrix();
mat4 persp_proj = perspective(camera.FOV, (float)Help::width / (float)Help::height, 0.1f, 1000.0f);
shaders["base"]->setMat4("view", view);
shaders["base"]->setMat4("proj", persp_proj);
```

##### processMovement
This function will process camera movement throughout our 3D space. Before we write this, let's return to the `main.cpp` function and add the following code:
```cpp
void specKeyPress(int key, int x, int y) {
	if (key == GLUT_KEY_SHIFT_L) { camera.DOWN = true; }
}

void specKeyUp(int key, int x, int y) {
	if (key == GLUT_KEY_SHIFT_L) { camera.DOWN = false; }
}

void keypress(unsigned char key, int x, int y) {
	if (key == ' ') camera.UP = true;
	if (key == 'w' || key == 'W') camera.FORWARD = true;
	if (key == 's' || key == 'S') camera.BACK = true;
	if (key == 'a' || key == 'A') camera.LEFT = true;
	if (key == 'd' || key == 'D') camera.RIGHT = true;
}

void keyup(unsigned char key, int x, int y) {
	if (key == ' ') camera.UP = false;
	if (key == 'w' || key == 'W') camera.FORWARD = false;
	if (key == 's' || key == 'S') camera.BACK = false;
	if (key == 'a' || key == 'A') camera.LEFT = false;
	if (key == 'd' || key == 'D') camera.RIGHT = false;
}
```

What do these functions do?
- `specKeyPress` handles the pressing of special keys (CTRL, SHIFT, ALT, etc.). Here we use the left shift key to descend.
- `specKeyUp` handles the release of special keys
- `keypress` handles the pressing of regular keys (alphanumeric, etc.). We use this function to control x- and z-axis movement using WASD and the y-axis using SPACEBAR.
- `keyup` handles the release of regular keys

We pass these functions into GLUT:
```cpp
glutIgnoreKeyRepeat(true); // ignore key held
glutDisplayFunc(display); // display scene
glutIdleFunc(updateScene); // update scene
glutKeyboardFunc(keypress); // key down
glutKeyboardUpFunc(keyup); // key up
glutSpecialFunc(specKeyPress); // special key down
glutSpecialUpFunc(specKeyUp); // special key up
glutPassiveMotionFunc(mouseMoved); // non-dragged mouse moved
glutSetCursor(GLUT_CURSOR_NONE); // hide cursor
```

We also add the `glutIgnoreKeyRepeat` function to ignore when the key is being held. If we had this enabled and didn't use bools to track keyboard presses, there would be movement, then a pause, then constant movement as GLUT realises we're holding down the key. This looks janky, so we don't do that.

Now, we get to the `processMovement` function. This function processes movement, obviously.
```cpp
if (FORWARD) {
	pos += normalise(front) * speed * Help::delta;
}
if (BACK) {
	pos -= normalise(front) * speed * Help::delta;
}
if (LEFT) {
	pos -= normalise(cross(front, up)) * speed * Help::delta;
}
if (RIGHT) {
	pos += normalise(cross(front, up)) * speed * Help::delta;
}
if (UP) {
	pos += vec3(0, speed * Help::delta, 0);
};
if (DOWN) {
	pos -= vec3(0, speed * Help::delta, 0);
};
```

`speed` is our movement speed. We multiply our speed by delta and the direction we're looking in to get our new position. When moving to the left or right, we get the cross product of our facing direction and our camera's up direction (NOT the global up) to get our right direction.

Finally, add the following line to the `updateScene` function in your `main.cpp` file:
```cpp
camera.processMovement();
```

And there you have it. You should be able to move freely throughout your scene, and look anywhere and everywhere you want.

#### Full Code
##### Camera.h
```cpp
#pragma once
#include "maths_funcs.h"
#include "help.h"
#include <GL/freeglut.h>

class Camera
{
public:
	Camera() {
		pos = vec3(0.0f);
		front = vec3(0.0f, 0.0f, -1.0f);
		up = vec3(0, 1, 0);
	}
	void processView(int, int);
	void processMovement();
	mat4 getViewMatrix();

	vec3 pos;
	vec3 front;
	vec3 up;
	vec3 tUP = vec3(0.0f, 1.0f, 0.0f);
	vec3 right;
	float FOV = 60.0f;
	float pitch;
	float yaw;
	float roll;
	float sensitivity = 15.0f;
	float speed = 5.0f;

	// Keyboard movement
	bool FORWARD = false;
	bool BACK = false;
	bool LEFT = false;
	bool RIGHT = false;
	bool UP = false;
	bool DOWN = false;
};
```

##### Camera.cpp
```cpp
#include "Camera.h"

void Camera::processView(int x, int y) {
	float xPos = Help::width / 2.0;
	float yPos = Help::height / 2.0;
	pitch = Help::clamp(pitch + ((y - yPos) * -sensitivity * delta), -89.0f, 89.0f);
	yaw = Help::wrap(yaw + ((x - xPos) * sensitivity * delta), 0.0f, 359.0f);
	glutWarpPointer(xPos, yPos);

	front = normalise(vec3(
		cos(Help::deg2Rad(yaw)) * cos(Help::deg2Rad(pitch)),
		sin(Help::deg2Rad(pitch)),
		sin(Help::deg2Rad(yaw)) * cos(Help::deg2Rad(pitch))
	));
	
	right = normalise(cross(front, tUP));
	up = normalise(cross(right, front));
}

void Camera::processMovement() {
	if (FORWARD) {
		pos += normalise(front) * speed * Help::delta;
	}
	if (BACK) {
		pos -= normalise(front) * speed * Help::delta;
	}
	if (LEFT) {
		pos -= normalise(cross(front, up)) * speed * Help::delta;
	}
	if (RIGHT) {
		pos += normalise(cross(front, up)) * speed * Help::delta;
	}
	if (UP) {
		pos += vec3(0, speed * Help::delta, 0);
	};
	if (DOWN) {
		pos -= vec3(0, speed * Help::delta, 0);
	};
}

mat4 Camera::getViewMatrix() {
	return look_at(pos, pos + front, up);
}
```



## Part 3 - Lighting
Lighting is another integral part of graphics. It lets you control things like colours, shadows, and ambient effects.

I used LearnOpenGL's method of providing lighting (which is somewhat simplistic). You can read up on this from the following links:
- https://learnopengl.com/Lighting/Colors
- https://learnopengl.com/Lighting/Basic-Lighting
- https://learnopengl.com/Lighting/Materials
- https://learnopengl.com/Lighting/Lighting-maps
- https://learnopengl.com/Lighting/Light-casters
- https://learnopengl.com/Lighting/Multiple-lights

(Yes, this is the entirety of his notes on OpenGL lighting. I've given you this much, don't get mad at me.)

#### Fragment Shader (Updated)
Your new fragment shader will look like this:
```cpp
#version 330
out vec4 FragColor;

struct Material {
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
}; 

struct DirLight {
    vec3 direction;
	
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

struct PointLight {
    vec3 position;
    
    float constant;
    float linear;
    float quadratic;
	
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

struct SpotLight {
    vec3 position;
    vec3 direction;
    float cutOff;
    float outerCutOff;
  
    float constant;
    float linear;
    float quadratic;
  
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;       
};

#define NR_POINT_LIGHTS 4

in vec3 FragPos;
in vec3 Normal;
in vec2 TexCoords;
// flat in int instanceID;

uniform vec3 viewPos;
uniform DirLight dirLight;
uniform PointLight pointLights[NR_POINT_LIGHTS];
uniform SpotLight spotLight;
uniform Material material;

// function prototypes
vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir);
vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir);
vec3 CalcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir);

void main()
{    
    // properties
    vec3 norm = normalize(Normal);
    vec3 viewDir = normalize(viewPos - FragPos);
    
    // == =====================================================
    // Our lighting is set up in 3 phases: directional, point lights and an optional flashlight
    // For each phase, a calculate function is defined that calculates the corresponding color
    // per lamp. In the main() function we take all the calculated colors and sum them up for
    // this fragment's final color.
    // == =====================================================
    // phase 1: directional lighting
    vec3 result = CalcDirLight(dirLight, norm, viewDir);

    // phase 2: point lights
    for(int i = 0; i < NR_POINT_LIGHTS; i++)
        result += CalcPointLight(pointLights[i], norm, FragPos, viewDir);    

    // phase 3: spot light
    result += CalcSpotLight(spotLight, norm, FragPos, viewDir);    
    
    FragColor = vec4(result, 1.0);
}

// calculates the color when using a directional light.
vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir)
{
    vec3 lightDir = normalize(-light.direction);
    // diffuse shading
    float diff = max(dot(normal, lightDir), 0.0);
    // specular shading
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    // combine results
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    return (ambient + diffuse + specular);
}

// calculates the color when using a point light.
vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
    vec3 lightDir = normalize(light.position - fragPos);
    // diffuse shading
    float diff = max(dot(normal, lightDir), 0.0);
    // specular shading
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    // attenuation
    float distance = length(light.position - fragPos);
    float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * (distance * distance));    
    // combine results
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    ambient *= attenuation;
    diffuse *= attenuation;
    specular *= attenuation;
    return (ambient + diffuse + specular);
}

// calculates the color when using a spot light.
vec3 CalcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
    vec3 lightDir = normalize(light.position - fragPos);
    // diffuse shading
    float diff = max(dot(normal, lightDir), 0.0);
    // specular shading
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    // attenuation
    float distance = length(light.position - fragPos);
    float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * (distance * distance));    
    // spotlight intensity
    float theta = dot(lightDir, normalize(-light.direction)); 
    float epsilon = light.cutOff - light.outerCutOff;
    float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);
    // combine results
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    ambient *= attenuation * intensity;
    diffuse *= attenuation * intensity;
    specular *= attenuation * intensity;
    return (ambient + diffuse + specular);
}
```

### Changes
In `main.cpp`, we will need to modify the uniform structs in our shader. Your `display` function will still use the shader you created, but now you can add a bunch of extra lights:
```cpp
// define this here, outside this function at the top of the file, or in your main.h file
vector<vec3> pointLightPositions = {
	vec3(1.3f, -2.0f, -2.5f),
	vec3(10.6f,  2.3f, -2.4f),
	vec3(1.3f,  5.1f, -1.1f),
	vec3(0.0f,  1.0f, -3.0f)
};

shaders["base"]->use();

shaders["base"]->setVec3("viewPos", camera.pos);
shaders["base"]->setFloat("material.shininess", 64.0f);
shaders["base"]->setInt("material.diffuse", 0);
shaders["base"]->setInt("material.specular", 1);

shaders["base"]->setVec3("dirLight.direction", -0.2f, -1.0f, -0.3f);
shaders["base"]->setVec3("dirLight.ambient", vec3(0.5f));
shaders["base"]->setVec3("dirLight.diffuse", vec3(0.8f));
shaders["base"]->setVec3("dirLight.specular", 0.5f, 0.5f, 0.5f);

// point light 1
shaders["base"]->setVec3("pointLights[0].position", pointLightPositions[0] * spread);
shaders["base"]->setVec3("pointLights[0].ambient", vec3(0.05f));
shaders["base"]->setVec3("pointLights[0].diffuse", 0.0f, 0.8f, 0.8f);
shaders["base"]->setVec3("pointLights[0].specular", 1.0f, 1.0f, 1.0f);
shaders["base"]->setFloat("pointLights[0].constant", 1.0f);
shaders["base"]->setFloat("pointLights[0].linear", 0.09f);
shaders["base"]->setFloat("pointLights[0].quadratic", 0.032f);
// point light 2
shaders["base"]->setVec3("pointLights[1].position", pointLightPositions[1] * spread);
shaders["base"]->setVec3("pointLights[1].ambient", vec3(0.05f));
shaders["base"]->setVec3("pointLights[1].diffuse", 0.0f, 0.8f, 0.8f);
shaders["base"]->setVec3("pointLights[1].specular", 1.0f, 1.0f, 1.0f);
shaders["base"]->setFloat("pointLights[1].constant", 1.0f);
shaders["base"]->setFloat("pointLights[1].linear", 0.09f);
shaders["base"]->setFloat("pointLights[1].quadratic", 0.032f);
// point light 3
shaders["base"]->setVec3("pointLights[2].position", pointLightPositions[2] * spread);
shaders["base"]->setVec3("pointLights[2].ambient", vec3(0.05f));
shaders["base"]->setVec3("pointLights[2].diffuse", 0.0f, 0.8f, 0.8f);
shaders["base"]->setVec3("pointLights[2].specular", 1.0f, 1.0f, 1.0f);
shaders["base"]->setFloat("pointLights[2].constant", 1.0f);
shaders["base"]->setFloat("pointLights[2].linear", 0.09f);
shaders["base"]->setFloat("pointLights[2].quadratic", 0.032f);
// point light 4
shaders["base"]->setVec3("pointLights[3].position", pointLightPositions[3] * spread);
shaders["base"]->setVec3("pointLights[3].ambient", vec3(0.05f));
shaders["base"]->setVec3("pointLights[3].diffuse", 0.0f, 0.8f, 0.8f);
shaders["base"]->setVec3("pointLights[3].specular", 1.0f, 1.0f, 1.0f);
shaders["base"]->setFloat("pointLights[3].constant", 1.0f);
shaders["base"]->setFloat("pointLights[3].linear", 0.09f);
shaders["base"]->setFloat("pointLights[3].quadratic", 0.032f);

shaders["base"]->setVec3("spotLight.position", camera.pos);
shaders["base"]->setVec3("spotLight.direction", camera.front);
shaders["base"]->setVec3("spotLight.ambient", vec3(0.05f));
shaders["base"]->setVec3("spotLight.diffuse", vec3(0.8f));
shaders["base"]->setVec3("spotLight.specular", vec3(1.0f));
shaders["base"]->setFloat("spotLight.constant", 1.0f);
shaders["base"]->setFloat("spotLight.linear", 0.09f);
shaders["base"]->setFloat("spotLight.quadratic", 0.032f);
shaders["base"]->setFloat("spotLight.cutOff", cos(Help::deg2Rad(2.5f)));
shaders["base"]->setVec3("spotLight.outerCutOff", cos(Help::deg2Rad(2.6f)));
```

This will create a flashlight effect using the spotlight to follow the camera's direction. 
## The End
That's everything, I think. Obviously, we are missing shadows and animation, and the Blender tutorial only shows you how to map textures instead of making new meshes, but this was meant as more of a starting point for you to learn than code for you to copy. Of course, you can just copy the code, but you should be mindful of things like comments and variable names. 

Thank you for reading. I hope you will enjoy using this tutorial as a guide for your final project, and if not, go find someone else's guide and use that. Why are you even here? Go away.

By Iris Onwa

### Sources
- https://learnopengl.com/
	- Shader class, lighting, Camera class, container2.jpg
- https://ogldev.org/
	- Mesh class, Texture class

