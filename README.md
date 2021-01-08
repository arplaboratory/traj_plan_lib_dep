Traj_generation uses various Quadratic programming 
==================================================================================

Run Demo
------------------------

This is a library containing various forms of trajectories In order to use this simply run the follow demo.

      catkin build traj_gen
      roslaunch traj_gen traj_gen.launch

Then in rviz open the Douple_path_config.rviz to visualize the following NLOPT and Eigen trajectories.

Library Features
------------------------

Folder include contains the .h files it is divided into 3 folders. 
  *  ooqp_interface - contains the .h files for interacting with OOQP - OoqpEigenInterface.hpp is the main one ot look at
  *  trajectory - contains the various trajectory formulations
  *  traj_utils - contains various utility functions for MSG passing and path visualization in RVIZ

Trajectories 
  *  Nlopt_traj -> Slowest trajectory generation uses the nonlinear optimization mostly as a proof of concept
  *  QPpolyTraj.h -> Contains 2 solvers for a QP formulation of the function of a polynomial based trajectories
        *  Fast Solve -> Solves 2x ->10x faster than Normal Solve but creates a slightly less optimal path
        *  Solve -> Basic Solve that finds the optimal path that minimizes snap for a polynomial
All Trajectories are solved in the same way as follows

	QPpolyTraj  qp_traj(dim); //integer for the dimensionality of each point x,y,z ->3 dim x,y,x,yaw
	qp_traj.setPolyOrder(polyOrder); //Set the polynomial Order. It should be greater or equal tot 10. 
      //Push various waypoints sequentially into the trajectory.
      //Waypoint class has 5 possible constraints, pos, velocity,acceleration, jer, and snap
            for(int i = 0; i < numPoints; i++){
                  Eigen::VectorXd v1(dim);
                  for(int j=0; j < dim; j++){
                        v1[j] = rand()%10-5;
                  }
                  //Creates a waypoint object with position constraint of v1
                  waypoint w = waypoint(v1);
                  nlopt_traj.push_back(w);
            }
     // auto generate the new time segment based on duration. Example below is 5
     //you must either generate times with autogenTimeSegment
     qp_traj.autogenTimeSegment(5);
     //Or Set time with the equivilient of qp_traj.setTimeSement(vectorXD timeSegments)
     //If you know the time you can also set the trajectory time previously
     //The coeffQP will contain the polynomial coefficients
     //This returns a Matrix of coefficients
     //Assume a polyOrder of 10
     // coeff(0,2) is the zeroth dimension polynomial, first segment, and its second order coefficients
     // coeff(1,2) is the first dimension polynomial, first segment, and its second order coefficients
     // coeff(1,15) is the first dimension polynomial, second segment, and its fifth order coefficients
     // Matrix is size numberDimension x polyOrder*number of Dimensions;
     Eigen::MatrixXd coeffQP =  qp_traj.solve(4); //Solve trajectory to minimize the order derivative 4 == Snap 1 == velocity.



Installation Dependencies
------------------------


1. Download and extract MA27
  *  http://www.hsl.rl.ac.uk/archive/
  *  Scroll down to MA27 - Note you don't need MA57

2. download OOQP and extract libariries
  * https://github.com/arplaboratory/traj_plan_lib_dep
  
            
3.Build MA27 with the following commands.

        ./configure FFLAGS="-O -fPIC"
        make clean 
        export CXXFLAGS="-O -fPIC"
        make -j4
        sudo make install

4.Set your enviorment variables for MA27
  *  NOTE: IT IS UNNECESSARY TO SET THE BLAS ENVIROMENT VARIABLE. The computer will naturally use the -lbas and -llpack
                                  
                MA27LIB=usr/local/lib/libma27.a 
                export MA27LIB

5. Download and install BLAS and LAPACKE

            sudo apt-get install libblas-dev liblapack-dev
            
6. Build OOQP using the following commands. : 

        ./configure FFLAGS="-O -fPIC"
        make clean 
        export CXXFLAGS="-O -fPIC"
        make -j4
        sudo make install

7. Gitpull this to your catkin directory and catkin build traj_gen to make this directory


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

