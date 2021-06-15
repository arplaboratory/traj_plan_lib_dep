Installation Dependencies
------------------------
If you are using Ubuntu 18.04, you should install gfortran compiler as well. 
```
$ sudo apt-get install gfortran
```

1.  extract MA27
  *  Source http://www.hsl.rl.ac.uk/archive/
  *  It is in the repo directory directory. You dont need to download it expliclty 
  * sudo apt-get install unzip 
  * unzip path_to_traj_plan_dep/ma27.zip

2.Build MA27 with the following commands. got o the extracted ma27 library

        ./configure FFLAGS="-O -fPIC"
        make clean 
        export CXXFLAGS="-O -fPIC"
        make -j4
        sudo make install
        
  *if there is configure error configure: error: cannot run /bin/bash ./config.sub, try run autoreconf --install

3.Set your enviorment variables for MA27
  *  NOTE: IT IS UNNECESSARY TO SET THE BLAS ENVIROMENT VARIABLE. The computer will naturally use the -lbas and -llpack
                                  
                MA27LIB=/usr/local/lib/libma27.a 
                export MA27LIB

4. Download and install BLAS and LAPACKE

            sudo apt-get install libblas-dev liblapack-dev

            
5. Build OOQP using the following commands. This should be done in the traj_plan_lib_dep directory where the ReadMe is in: 

        ./configure FFLAGS="-O -fPIC"
        make clean 
        export CXXFLAGS="-O -fPIC"
        make -j4
        sudo make install

Sometimes you may have the following issue configure: error: cannot guess build type; you must specify one
Please refer to the following link on how to resolve the issue.
https://stackoverflow.com/questions/4810996/how-to-resolve-configure-guessing-build-type-failure


# traj_plan_lib_dep

1. This came from this link download OOQP and extract libariries
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

