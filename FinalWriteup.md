# CSE 168 Final - Fluid Simulation and Volumetric Rendering
## Luke Henry

For my final project, I implemented a semi-lagrangian fluid simulator to render smoke using volumetric rendering. The renderer uses a SmokeField class that contains a 3D vector field of flow velocities and scalar fields for pressure, temperature, and smoke density. The fields change based on the Incompressible Navier Stokes equation, with added external forces for buoyancy, gravity, and vortex confinement to correct for the inconsistencies and inacuracies of semi-lagrangian advection and the assumtion of constant density. The simulator enables the pathtracer to render multiple frames as the smoke fields change over small time steps, which have then been compiled into gifs outside of the renderer for easier viewing. 

I found this paper from Stanford describing a particular method for rendering smoke that has sources that link to the basics of fluid simulation as well, which I have been referencing with respect to the math. I tried to use some of the same methods and equations used in the paper, including the Conjugate Gradient method for solving the pressure equation, monotonic cubic interpolation instead of trilinear interpolation, and the Henyey-Greenstein function for the phase probability in my volumetric rendering method. 
[Fedkiw et al](https://web.stanford.edu/class/cs237d/smoke.pdf)


Semi-Lagrangian advection is a method for simulating fluid flow that uses a grid of flow velocities. It relies on the assumptions that all relevant fields follow the flow velocity field (are advected through it) and that the velocity at a given point does not change much over the course of a time step. It is used to generate and update a discrete grid of points based on estimates where an analytical or continuous solution to the Navier Stokes equation is unstable or non-existent. 

![1](https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_Advection_random.gif?raw=true)

![2](https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_GravBuoy_Rise.gif?raw=true)

Both of the gifs above demonstrate the effects of advection, where the smoke moves along with the velocity field. The second gif shows viscosity, gravity and bouyancy forces applied as well. The main reason it does not look like smoke is because it lacks the pressure correction term from the Navier Stokes equation for incompressible fluids.

The Navier Stokes Equation for Incompressible Fluids with constant viscosity is:

![Nevier Stokes Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/e5e8521f648a2a1f7525f4f0dd166bbfbb079b0f)

(source: wikipedia)

Where `u` is velocity, `f` is external force, `p` is pressure, `ρ` is density which can be set to 1 as it is constant and unitless, and `v` is viscosity which can also be constant and very low, such as 0.001 for gasses like air and smoke.

The main assumption for incompressible fluids is that the divergence of the velocity field at any point is 0, so that density is constant. The main term in the equation is the gradient of the pressure field, and in order for this assumption to hold, the following Poisson Equation for pressure must be solved to find the correct pressure field values:

![Poisson Pressure Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/9a0d9f8b11680878c6fe4cd016eb5e780ee1d980)

(source: wikipedia)

This process involves solving an extremely large linear matrix equation `Ap = b` where A is a sparce matrix that has many 0's. Again, an analytical solution would be far too complex and expensive to compute, so an iterative solver is used instead. In this case, I used the Conjugate Gradient method, which is supposed to converge faster than many other methods for large, symmetric matrices. The matrix `A` in this context is a matrix relating each cell to its 6 nearest neighbors. It contains a row for every pressure value in the field, but most of each row is 0. Multiplying it by a field ultimately means taking a rescaled, weighted sum of the value at each cell with the values at its directly neighboring cells. The conjugate gradient method involves calculating a search direction and step size based on a residual from `b`, where `b` is the divergence of the velocity field, then taking small steps in the search direction and refining each value over multiple iterations. 

With the solver applied (and simple linear interpolation between grid points), we get results like this:

<img src="https://github.com/LukeHenry04/CSE168_websites/blob/main/Smoke_bottom_cg_lerp.gif?raw=true" alt="Alt Text" width="200" height="200">

When sampling a point within the grid, interpolation is used to blend between corners of each voxel. Linear interpolation, shown above, works but ends up blending away a lot of finer details. I implemented the monotonic cubic interpolation used in the Standford paper above. Unlike linear interpolation that requires 2 input points, cubic interpolation requires 4. In 3 dimensions this results in 64 grid points being used for interpolation at each sample. The performance slowdown is very noticable, but the quality of the smoke even without volumetric rendering is noticably sharper with cubic interpolation enabled:

<img src="https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_bottomCubicInterpolation.gif?raw=true" alt="Alt Text" width="200" height="200">

After implementing the main fluid simulator, I also added a vortex confinement force, also as described in the Stanford paper above, which tries to add back some of the spiraling turbulent patterns that are lost between the interpolation, advection approximations, and finite resolution grid. This force is proportional to the gradient of the curl of the velocity field, and added in makes the simulation look like this:

<img src="https://github.com/LukeHenry04/CSE168_websites/blob/main/LerpVortex.gif?raw=true" alt="Alt Text" width="400" height="400"> (rendered with darker smoke and linear interpolation)

Once the simulation was running well, the last feature I implemented was volumetric rendering. The raytracer uses an integrater called "smokeOnly" which marches along each ray from the camera and calculates monte carlo direct lighting for each point along the main ray. It does this by sending out rays to each quadlight light source (stratified and sampled using the same techniques as in the monte carlo homework), but instead of using a brdf, it first integrates the smoke field along the ray to get an amount of smoke between the point and the light. It plugs this smoke occlusion amount into `e^(-x)` to get a transmittance amount. It then scales it by a phase probabilty (for forward scattering effects) obtained from the Henyey-Greenstein function described in the Stanford paper. The function is loosely inversely proportional to the cosine between the direction to the light source and the direction of the main ray from the camera. It also takes in a constant g between 0 and 1, which is based on the amount of isotropic scattering as opposed to forward directional scattering. (All of the images below use `g=0.5`). The product of the light source color, transmittance, and phase term gives the amount of light scattered towards the camera at each point. The amount of light scattered is also scaled by the amount of smoke in the field at the point being sampled, since that determines transparency vs opaqueness. 

The last step is to find the transmittance between the camera and the point along the ray, and scale the amount of scattered light down by that amount (plus a material scattering constant and the step size), to get the sum of the incoming light along the entire ray, which is outputted to the pixel color. 

Here are some of the resulting images and gifs:

![F1](https://github.com/LukeHenry04/CSE168_websites/blob/main/PlumeLerp.gif?raw=true)

![F2](https://github.com/LukeHenry04/CSE168_websites/blob/main/PlumeWindRightwardLerp.gif?raw=true)

![F3](https://github.com/LukeHenry04/CSE168_websites/blob/main/VR_lerp_floor_coarseGrid.gif?raw=true) ![F3_1](https://github.com/LukeHenry04/CSE168_websites/blob/main/SmokeFloor1.png?raw=true)

(Note that the above 3 use linear interpolation for longer animations, as rendering multiple frames is significantly more expensive with cubic interpolation, which again uses 64 sample points as opposed to 8. Also the last animation has artifacts from the large raymarching steps, but I figured it was worth including)

Here is a selection of frames from the second animation, rendered with cubic interpolation:

![F4](https://github.com/LukeHenry04/CSE168_websites/blob/main/CubicPlume.gif?raw=true)

![F5](https://github.com/LukeHenry04/CSE168_websites/blob/main/20smoke.png?raw=true) ![F6](https://github.com/LukeHenry04/CSE168_websites/blob/main/21smoke.png?raw=true) ![F7](https://github.com/LukeHenry04/CSE168_websites/blob/main/22smoke.png?raw=true)

Here is the frame before rendered with linear interpolation to show the difference in fine details:

![F8](https://github.com/LukeHenry04/CSE168_websites/blob/main/19smoke.png?raw=true) 


(Lastly, for grading, the vast majority of the changes to the raytracer are in the two new files SmokeField.h and SmokeField.cpp, with a minimal amount of changes to Raytracer.cpp::main which handles multi-frame renders and implements volumetric rendering and the main fluid simulation steps through calls to methods in SmokeField.cpp)







