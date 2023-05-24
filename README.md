# Perception System

Eagle Trento Racing Team driverless team.

System gets stereo images, detects cones in it, generates a map in which selects points of the cones, filters the map keeping only one point per cone, then serializes the clean map and trajectory and sends it to whoever needs it.


## Requirements


## Installation
~~~bash
mkdir build
cd build
cmake ..
make
~~~

## Executables
### Logger
Script that gets stereo images and IMU data and saves them.  
Is used to create datasets to be later processed.


## Convention
### Functions
~~~c++
class UpperCase
{
public:
  /**
  * Comment of function
  * @param param_name description of param
  * @return return random float
  */
   double FunctionIsPublic();
private:
  // variable comment
  int variable_one;
};

// instance of class
UpperCase mapOptimizer;

int variable_outside_class;
const double k_is_constant;

void functionOutsideClass()
{
  // in function comment
}

typedef struct ThisIsAStruct
{
  char inside_struct;
}

enum Number{
  ONE,
  TWO,
  THREE
}
~~~
