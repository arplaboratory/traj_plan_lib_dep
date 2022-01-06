# traj_plan_lib_dep
This is the dependencies repo for the trajectory generation package in the ARPL. 

If you have any question regarding the repo, please feel free to post questions in the Issues or ask the maintainer directly. 

**Developer: Jeffery Mao<br />
Affiliation: [NYU ARPL](https://wp.nyu.edu/arpl/)<br />
Maintainer: <br />
Jeffery Mao, jm7752@nyu.edu<br />
Guanrui Li, lguanrui@nyu.edu<br />**

### Prerequsite
```
$ sudo apt-get install gfortran
$ sudo apt-get install unzip 
```

### Installation
0. Clone this repo to your workspace
```
$ cd /path/to/your/workspace/src
$ git clone git@github.com:arplaboratory/traj_plan_lib_dep.git
$ cd traj_plan_lib_dep
```
1.  Install MA27 (source: http://www.hsl.rl.ac.uk/archive/)
    
    The MA27 package is already in this repo directory directory.  You can just unzip the ma27.zip like following:
```
$ unzip ma27.zip
```

Then you can build MA27 with the following commands. 
```
$ cd ma27
$ ./configure FFLAGS="-O -fPIC"
$ make clean 
$ export CXXFLAGS="-O -fPIC"
$ make -j4
$ sudo make install
```        
   *  NOTE: if there is configure error configure: error: cannot run /bin/bash ./config.sub, try to run 
```   
$ autoreconf --install
``` 

If autoreconf --install failed then run  

```   
$ ./configure --build=aarch64-unknown-linux-gnu
``` 
	 
Now you should set your enviorment variables for MA27
```
$ export MA27LIB=/usr/local/lib/libma27.a 
```                                         
  *  NOTE: IT IS UNNECESSARY TO SET THE BLAS ENVIROMENT VARIABLE. The computer will naturally use the -lbas and -llpack
  
2. Download and install BLAS and LAPACKE
```
$ sudo apt-get install libblas-dev liblapack-dev
```

3. Build OOQP using the following commands. This should be done in the traj_plan_lib_dep directory where the ReadMe is in: 
```
$ cd /path/to/traj_plan_lib_dep
$ ./configure FFLAGS="-O -fPIC"
$ make clean 
$ export CXXFLAGS="-O -fPIC"
$ make -j4
$ sudo make install
```   
Sometimes you may have the following issue configure: error: cannot guess build type; you must specify one.
For Nvidia jetson TX2 you can resolve this issue with 
```
$ ./configure --build=aarch64-unknown-linux-gnu FFLAGS="-O -fPIC"
```
Please refer to the following link for more details on how to resolve the issue.
https://stackoverflow.com/questions/4810996/how-to-resolve-configure-guessing-build-type-failure

## INSTALLING on Dragonfly
1. For Trajectory Planner. Remember to pull from the ``main`` branch in order to install on the Dragonfly.
https://github.com/arplaboratory/ARPL_Trajectory_Planning

## Additional Information about the changes we made in the OOQP

1. We download OOQP for the following link and extract libariries
  *  https://github.com/emgertz/OOQP/releases/tag/OOQP-0.99.27
  
2. In $OOQP_Directory/src/Vector/SimpleVector.c We replaced the lines 221-234 in the following code snippet

          void SimpleVector::scale( double alpha )
          {
            int one = 1;
            dscal_( &n, &alpha, v, &one ); 
          }

          void SimpleVector::axpy( double alpha, OoqpVector& vec )
          {
            assert( n == vec.length() );
            SimpleVector & sv = dynamic_cast<SimpleVector &>(vec);

            int one = 1;
            daxpy_( &n, &alpha, sv.v, &one, v, &one );
          }


With the following

     void SimpleVector::scale( double alpha )
     {
       int i;
       for( i = 0; i < n; i++ ){
        v[i] = alpha*v[i];
       }
     }

     void SimpleVector::axpy( double alpha, OoqpVector& vec )
     {
       assert( n == vec.length() );
       SimpleVector & sv = dynamic_cast<SimpleVector &>(vec);
       int i = 0;
       for( i = 0; i < n; i++ ){
        v[i] += alpha*sv[i];          
            }
      }
