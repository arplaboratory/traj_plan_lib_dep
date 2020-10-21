# traj_plan_lib_dep

1. This came from this link download OOQP and extract libariries
  *  https://github.com/emgertz/OOQP/releases/tag/OOQP-0.99.27
  
  
2. In $OOQP_Directory/src/Vector/SimpleVector.c lines 221-234 replace the following code snippet

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

