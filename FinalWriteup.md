# CSE 168 Final - Fluid Simulation and Volumetric Rendering
## Luke Henry

For my final project, I implemented a semi-lagrangian fluid simulator to render smoke using volumetric rendering. The project uses a SmokeField class that contains a 3D vector field of flow velocities and scalar fields for pressure, temperature, and smoke density. The fields change based on the Incompressible Navier Stokes equation, with added external forces for buoyancy, gravity, and vortex confinement to correct for the inconsistencies and inacuracies of semi-lagrangian advection and the assumtion of constant density. The simulator allows enables the pathtracer to render multiple frames as the smoke fields change over small time steps, which have then been compiled into gifs. 

I found this paper from Stanford describing a particular method for rendering smoke that has sources that link to the basics of fluid simulation as well, which I have been referencing with respect to the math. I tried to use some of the same methods and equations used in the paper, including the Conjugate Gradient method for solving the pressure equation, monotonic cubic interpolation instead of trilinear interpolation, and the Henyey-Greenstein function for the phase probability in my volumetric rendering method. 
[Fedkiw et al](https://web.stanford.edu/class/cs237d/smoke.pdf)


Semi-Lagrangian advection is a method for simulating fluid flow that uses a grid of flow velocities. It relies on the assumptions that all relevant fields follow the flow velocity field (are advected through it) and that the velocity at a given point does not change much over the course of a time step. It is used to generate and update a discrete grid of points based on estimates where an analytical or continuous solution to the Navier Stokes equation is unstable or non-existent. 

The Navier Stokes Equation for Incompressible Fluids with constant viscosity is:

![Nevier Stokes Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/e5e8521f648a2a1f7525f4f0dd166bbfbb079b0f)

(source: wikipedia)

Where `p` is pressure, `ρ` is density which can be set to 1 as it is constant and unitless, and `v` is viscosity which can also be constant and very low, such as 0.001 for gasses like air and smoke.

The main assumption for incompressible fluids is that the divergence of the velocity vield at any point is 0, so that density is constant. The main term in the equation is the Gradient of the pressure field, and in order for this assumption to hold, the following Poisson Equation for pressure must be solved to find the correct pressure field values:

![Poisson Pressure Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/9a0d9f8b11680878c6fe4cd016eb5e780ee1d980)

(source: wikipedia)

This process involves solving an extremely large linear matrix equation `Ap = b` where A is a sparce matrix that has many 0's. Again, an analytical solution would be far too complex and expensive to compute, so an iterative solver is used instead. In this case, I used the Conjugate Gradient method, which is supposed to converge faster than many other methods for large, symmetric matrices. 

It is also important to note that sampling between grid points requires some form of interpolation for smoother results, which will also have to be implemented. 

Currently, my project has the following steps implemented:
- SmokeField class with main methods, variables, and fields
- The ability to render multiple frames and timesteps
- Semi-Lagrangian Advection
- The viscosity times Laplacian(velocity) term from Navier Stokes (implemented with discrete partial derivatives based on the limit definition except h is the spacing between grid points)
- Gravity force proportional to smoke density at each point
- Buoyancy force proportional to difference between a given point's temperature and a set ambient temperature
- A method to inject smoke into the SmokeField
- A simple method for integrating the amount of smoke along a ray and scaling it's radience down by the amount of smoke passed through

The rest of my project will involve the following major steps:
- Iterative pressure solver for Navier Stokes
- Vortex confinement external force (makes up for numerical instability and blurring from interpolation)
- Better volumetric rendering, like direct lighiting and shadows on the smoke.
- Either adding the volumetric rendering into regular pathtraced scenes, or focusing on getting higher quality smoke renders, including "collisions" with simple geometry
- Creating smoke emmiter objects that are handled collectively, emmiting with a radius, temperature, and smoke density as described in scene file commands


I have the following gifs demonstrating (in order) advection with a constant velocity field, advection with a random velocity field, buoyancy and gravity forces on injected smoke (note that the last two gifs contain the viscosity term, but since not much is happening and it is a small effect since smoke is not viscous, it does not show much change):

![1](https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_Advection.gif?raw=true)

![2](https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_Advection_random.gif?raw=true)

![3](https://github.com/LukeHenry04/CSE168_websites/blob/main/SMOKE_GravBuoy_Rise.gif?raw=true)


Also note that the pixelated nature of the images is due to a lack of interpolation between grid points, what I have rendered is the grid itself for now. 



