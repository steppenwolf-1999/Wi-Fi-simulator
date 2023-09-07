tips:The formula may not be in the right Latex form, so anyone want to read this in detail can try download this markdown file to your own computer

注：本文中公式的渲染方式可能在Github上不能正确展示，如果需要了解文章中的算法细节可以尝试把本markdown文件下载到本地阅读







# Wi-Fi Signal Strength Mapping Application Using Java and C++ with Java Native Interface

### Description
Wi-Fi signal strength mapping application is a project that aims to provide the most ideal spot to place the Wi-Fi router, ensuring the entirety of the house receives an adequate reception. The system enables users to enter their floor plan as a PNG image, modify certain parameters like frequency and material of the walls and then pick a spot to place the router, the system then takes those inputs and generates a heatmap image which displays the flow and reach of the signals. 



# Analysis and algorithms

### 1. Helmholtz equation

​	First let's explain the Helmholtz equation:

$$
\Delta E +\frac{k^2}{n^2}E=f
$$

​	which can be also written in two dimension expanded form:

$$
(\frac{\partial^2}{\partial^2 x}+\frac{\partial^2}{\partial^2 y})E(x,y)+\frac{k^2}{n(x,y)^2}E(x,y)=f(x,y)
$$

​	Here $E$ means the amplitude of the electric field (which can be considered as Wi-Fi strength), $n$ means the refraction index and $f$ means the wave source distribution.

​	To understand why our algorithms need to solve sparse matrix calculation, we first need to understand why there is $E(x,y)$, $n(x,y)$ and $f(x,y)$. In physics, if a quantity $X$ is followed by a bracket $(x,y)$, it means the quantity takes different value in different place in the $x-y$ plane. For example, $E(x,y)$ means the amplitude of the electric field is different in different place. $n(x,y)$ means the refraction index is different in different place. So as $f(x,y)$.

### 2. refraction index $n(x,y)$ and wavelength 

<font color='blue'>In[19]</font>.

```
η_air = 1.                 # refraction index for air
η_concrete = 2.55 - 0.01im  # refraction index for concrete
# the imaginary part conveys the absorption

λ = 0.12                  # for a 2.5 GHz signal, wavelength is ~ 12cm
k = 2π / λ                # k is the wavenumber
```

​	Here $n_{air}=1$ and $n_{con}=2.55-0.01i$ is reflection index. Notice that they are complex value. In physics without using Maxwell equation the refraction index is a  real number, which is always explained in this picture:

![image-20221110175926517](C:\Users\lzx\AppData\Roaming\Typora\typora-user-images\image-20221110175926517.png)

​	And we know $n=c/v$. But a real number refraction index only describe how an electric-magnetic wave is deflected in a material. But in fact if we take the Maxwell equation we can see the material will not only deflect the electric-magnetic wave, it also absorb part of it. And it is represented by the imaginary part of refraction index. So in our project $n$ is a complex value. And where there is air, $n_{air}=1$, which means the air do not deflect or absorb electric-magnetic wave. Where there is concrete wall, $n_{con}=2.55-0.01i$, which means the wall will deflect and absorb electric-magnetic wave. And you may notice:

<font color="blue">In[20]</font>

```
μ = similar(plan, Complex)
μ[plan .!= 0] = (k / η_air)^2
μ[plan .== 0] = (k / η_concrete)^2;
```

​	Here we are actually creating a matrix of $\frac{k^2}{n(x,y)^2}$ same size as our plan(360*224), where there is air $n=1$ and where there is concrete wall $n=2.55-0.01i$. The result is the whole refraction index metric is a complex metric, which will consume double time in matrix calculation.

​	Another issue to pay attention to is we can easily add <font color='red'>**wood material**</font> in to our simulation. Just add a new parameter $n_{wood}$ into the refraction index matrix. The only thing is we still do not know how to determine the parameter up to now.

### 3. How to change the Helmholtz equation into finite form

​	This part I will derive the finite form of Helmholtz equation. As shown before, the two dimension Helmholtz equation is:

$$
(\frac{\partial^2}{\partial^2 x}+\frac{\partial^2}{\partial^2 y})E(x,y)+\frac{k^2}{n(x,y)^2}E(x,y)=f(x,y)
$$

​	which can also be written as:

$$
\frac{\partial}{\partial x}(\frac{\partial E(x,y)}{\partial x})+\frac{\partial}{\partial y}(\frac{\partial E(x,y)}{\partial y})+\frac{k^2}{n(x,y)^2}E(x,y)=f(x,y)
$$

​	You may remember the partial derivative means how fast a function changes, so:

$$
\frac{\partial E(x,y)}{\partial x}=\lim\limits_{\Delta x\rightarrow\infty} \frac{E(x+\Delta x,y)-E(x,y)}{\Delta x}
$$

​	and

$$
\lim\limits_{\Delta x\rightarrow\infty} \frac{\partial E(x-\Delta x,y)}{\partial x}=\lim\limits_{\Delta x\rightarrow\infty} \frac{E(x,y)-E(x-\Delta x,y)}{\Delta x}
$$

​	So we have:


$$
\frac{\partial}{\partial x}(\frac{\partial E(x,y)}{\partial x})=\lim\limits_{\Delta x\rightarrow\infty} \frac{\frac{\partial E(x,y)}{\partial x}-\frac{\partial E(x-\Delta x,y)}{\partial x}}{\Delta x}\\ =\lim\limits_{\Delta x\rightarrow\infty}\frac{\frac{E(x+\Delta x,y)-E(x,y)}{\Delta x}-\frac{E(x,y)-E(x-\Delta x,y)}{\Delta x}}{\Delta x} \\=\lim\limits_{\Delta x\rightarrow\infty} \frac{E(x+\Delta x,y)+E(x-\Delta x,y)-2E(x,y)}{(\Delta x)^2}
$$

​	Then Helmholtz equation can be written as:

$$
\lim\limits_{\Delta x\rightarrow\infty} \frac{E(x+\Delta x,y)+E(x-\Delta x,y)-2E(x,y)}{(\Delta x)^2}+ \frac{E(x,y+\Delta y)+E(x,y-\Delta y)-2E(x,y)}{(\Delta y)^2}+\frac{k^2}{n(x,y)^2}E(x,y)=f(x,y)
$$

​	But differentiating a function to an infinite small is a conceptual idea, which can only be achieved in brain. Luckily physicians already proved that computer do the differentiation a bit at one time, the final result converge to the infinite form! Which means if we write the above equation in a finite form and set $\Delta x$ and $\Delta y$ to a very small value, the result given by computer will be very similar to the reality.

 	Then we get the final form of Helmholtz equation that computer can really solve, which is just to remove the $lim$ sign:

$$
\frac{E(x+\Delta x,y)+E(x-\Delta x,y)-2E(x,y)}{(\Delta x)^2}+ \frac{E(x,y+\Delta y)+E(x,y-\Delta y)-2E(x,y)}{(\Delta y)^2}+\frac{k^2}{n(x,y)^2}E(x,y)=f(x,y)
$$

​	And assume each step alone the $x$ or $y$ direction is just the length of one pixel, we can turn the scale of our finite differentiation exactly same as the scale of our input picture(360*224). That's why in the original blog the writer use $i$ and $j$ as index to $x$ and $y$ and get:

$$
\frac{E(i+1,j)+E(i-1,j)-2E(i,j)}{(\Delta x)^2}+ \frac{E(i,j+1)+E(i,j-1)-2E(i,j)}{(\Delta y)^2}+\frac{k^2}{n(i,j)^2}E(i,j)=f(i,j)
$$

​	Notice three thing: 

​	First, defining how many steps we differentiate alone x and y the same as our picture pixels number give our benefit that the following matrix calculation can carry out.

​	Second, the bigger number of steps we differentiate, the more accurate result we will get, but the number of step need to stay the same as number of pixels, and it also need to stay a relatively small number in order to make the calculation time acceptable. Luckily, our scale of 360*224 seems good enough.

​	Third, the above equation can be explained in a more intuitive way, the finite form describe how a point $(i,j)$ interact with its four neighbors:  $ (i+1,j)$ ,  $ (i-1,j)$ ,  $(i,j+1)$  and  $ (i,j-1)$ . In a graphic way to describe this, its like:

![image-20221111155642530](C:\Users\lzx\AppData\Roaming\Typora\typora-user-images\image-20221111155642530.png)

​	

### How to do the matrix calculation

#### please refer to both links in this part

https://nbviewer.org/gist/fredo-dedup/31ae1b6017833e9a18f8#Setting-the-up-the-data

https://jasmcole.com/2014/08/25/helmhurts/?from=groupmessage&isappinstalled=0&scene=1&clicktime=1584852482&enterid=1584852482



Java matrix library:

[Dense and Sparse Matrix Libraries for Java: An Overview (java-matrix.org)](https://java-matrix.org/)





JSci: up to now the best

​	What we need: a solver to linear system such as:

 $A \cdot X = b$
$$
X=A^{-1}*b
$$


​	and the goal is to get:

$X=A^{-1}*b$

​	if $A$ is a complex matrix, $b$ is a double vector, $X$ has to be a complex vector.



### Implementation
**A. C++ Compiler and Eclipse IDE Setup**
A C++ compiler such as MinGW is required to be installed in Windows operating systems. This compiler needs to be made available throughout the system by adding it to System Environment. MAC has GCC toolchains installed by default. Eclipse C/C++ IDE CDT needs to be installed to support development of C++ project.

**B. Eigen C++ library**
Eigen C++ template library for linear algebra(version 3.4.0) has to be downloaded, unzipped and added to an accessible location. The path to the Eigen headers folder in the downloaded package needs to be added to C++ compiler includes. Also, the includes headers path to JDK needs to be added to the C++ compiler includes.

**C. Signal Strength Computation Algorithm**
The core logic for Wi-Fi signal strength computation has been implemented using the Eigen Linear Algebra library. The complex sparse matrices use the SparseMatrix<double> class from SparseCore package. An array of triplets having sparse matrix coordinates and values is built. This is pushed into the sparse matrix in one operation using setFromTriplets function. This accelerated the sparse matrix construction process which was very slow otherwise with regular iteration. The sparse matrix linear equation was solved using SparseLU<SparseMatrix<complex<double>>> Decomposition solver and the resulting vector was reshaped into a two-dimensional image matrix using the reshaped function which is a column major vector to matrix converter.

**D. Java Native Interface (JNI)**
A class having the interfacing methods, EnergyComputationAlgoNative was created on the Java project. The interfacing function has the keyword ‘native’ to help java compiler understand that the function needs to be added to JNI bridging headers. External tools configuration needs to be added specifying the path to Java compiler, the destination of the bridging header file and the source file having the bridging functions with native keywords in the Java project. On running the external tools configuration, JNI bridging header file is generated. Implementation is written for this header on C++ project where the input parameter objects are mapped to C++ data structures. In our case, a two-dimensional double matrix, double values and integers were the parameters passed. This bridging function should also map the C++ data structures back to JNI objects while returning from the function.
Header files were also written on the C++ project for code segregation. The actual algorithm was written inside another C++ class called EnergyComputationAlgo. An instance object of this class was created and the function for computing was invoked from the JNI bridging function.
The C++ compiler needs to be configured to generate dll or dylib shared dynamic libraries (based on the OS) instead of the standard executable file. Also, NDEBUG flag has to be added to C++ pre-processor and -O3 optimisation needs to be added to the compiler flags in the C++ compiler settings. Upon building the C++ project, the generated dynamic library is saved to the configured destination.

**E. Java JNI Integration**
The generated C++ dynamic library is copied and added to the Java project workspace. This library is dynamically loaded using the System.load function. Whenever a call to the function with native keyword (in our case computeWifiEnergy) is made, Java dynamically loads this library and calls the matching function on the JNI bridging header. This in turn invokes the C++ function and we retrieve the results. The resulting matrix values are rescaled to a range of 1-100 in order to map them to different color values. A color map of hundred colors using a base color specified is created and final matrix is built by mapping this to the matrix. This matrix is converted back to an image by PixelWriter.

**F. Multithreading**
The energy computation algorithm is a heavy process as there is a matrix decomposition operation involved. This makes it a slow process and running it on main thread causes the UI to freeze. In order to make UX better, the heavy computation was switched to a background thread using the CalculationThread class which implements Runnable interface. Once the calculation operation is complete and a response is received from the C++ dynamic library, we have to continue the UI operation through a callback function. For this, a Callback interface is written and the class calling this thread operation(RouterController) implements the callback interface function. The callback function is invoked from inside Platform.runLater block to run the UI operations again on the main thread.

**G. JavaFX UI**
The UI is created using .fxml files and .java controller files. The fxml files are updated using Scene Builder by dragging and dropping various elements. These elements have been assigned an Id, and linked to certain controller methods at various events like mouse click. Saperate controller files are created corresponding to each page and each .fxml file. The .fxml files are linked to their respective controller files and a .css file to implement design of the application. A "Properties" class file is created which contains static data fields and methods, the different controller use the static getter and setter methods to update or retrieve the values present in the data fields. Finally, everything is wrapped using a scene and stage. The movement between different screens is achieved by updating the scenes with the root of different .fxml files and setting the stage with the new scene.

