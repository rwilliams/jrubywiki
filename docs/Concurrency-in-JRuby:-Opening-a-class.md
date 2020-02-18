Let us consider modifications to a class C's methods when it is opened (or reopened) concurrently by two threads T1 and T2.  Here are the operational guarantees that JRuby provides in this scenario (M, M1, and M2 represent methods):

1. If T1 adds M1 and T2 adds M2, JRuby guarantees that M1 and M2 will both be added no matter what order T1 and T2 run in and no matter how their execution is interleaved.
2. If T1 adds M and T2 removes M, JRuby does not (and cannot) guarantee that when they are done, M will be present or absent in class C.  This is a user-program error.
3. If T1 adds/removes M and T2 reads M, then JRuby does not (and cannot) guarantee that T2's access to M will OR will not result in an undefined-method error.  This is a user-program error.
4. If T1 adds M and T2 adds M, JRuby does not (and cannot) guarantee which version of M will be present in C, or that the same version of M will be consistently seen.  This is a user-program error.

So, JRuby does not guarantee a specific order in which the method updates will be performed to M.  This is the same as saying that the JRuby runtime cannot guarantee that the threads T1 and T2 will be scheduled in a specific order or that their instructions will be interleaved in a specific order (except where enforced by the program itself).

The JRuby runtime can be said to be not thread-safe only if it didn't handle scenario 1. above correctly and if at the end of execution of T1 and T2, one or both of M1 or M2 were missing from class C.