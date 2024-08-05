# acceleration_incompatibleWiththree-phase
 [Bug] and [Fix]: using the vanilar acceleration event with three-phase + elasticity does not work



## Potentially, the could be the reinitialization of the acceleration source term. 

### [src-local/log-conform-elastic.h](src-local/log-conform-elastic.h)

```c
event acceleration (i++)
{
  face vector av = a;
  foreach_face()
    if (fm.x[] > 1e-20) {
      double shear = (tau_p.x.y[0,1]*cm[0,1] + tau_p.x.y[-1,1]*cm[-1,1] -
		      tau_p.x.y[0,-1]*cm[0,-1] - tau_p.x.y[-1,-1]*cm[-1,-1])/4.;
      av.x[] += (shear + cm[]*tau_p.x.x[] - cm[-1]*tau_p.x.x[-1])*
	alpha.x[]/(sq(fm.x[])*Delta);
    }
#if AXI
  foreach_face(y)
    if (y > 0.)
      av.y[] -= (tau_qq[] + tau_qq[0,-1])*alpha.y[]/sq(y)/2.;
#endif
}
```

### [testGravity_noreduced.c](testGravity_noreduced.c)

```c
event acceleration(i++){
  face vector av = a;
  foreach_face(x){
    av.x[] += Bond;
  }
}
```

## Workaround (actually a feature)

Use [src-local/reduced-three-phase-nonCoalescing.h](src-local/reduced-three-phase-nonCoalescing.h). See [testGravity_Withreduced.c](testGravity_Withreduced.c).


### Left: acceleration method and Right: reduced gravity approach

<div style="display: flex; justify-content: space-between;">
  <div style="flex: 1; margin-right: 10px;">
    <video width="100%" controls>
      <source src="Video_Noreduced.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
  <div style="flex: 1; margin-left: 10px;">
    <video width="100%" controls>
      <source src="Video_Withreduced.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
  </div>
</div>

The drop at the center (blue) should have gone up as it is lighter than the surrounding and gravity is downwards. The video on the right is the correct implementation.

## Concluding remarks

Aug 5, 2024

In theory, codes [testGravity_noreduced.c](testGravity_noreduced.c) and [testGravity_Withreduced.c](testGravity_Withreduced.c). Exactly what part of the code is responsible for the bug is not clear. More investigation is needed.

**Note:** Although the code has elasticity (becuase I am unsure if the bug is a three-phase bug or a elastic bug--that will be the first thing to check), the elastic modulus (set by the elasto-capillary number $Ec$) is set to zero.