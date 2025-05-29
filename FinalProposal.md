# CSE 168 Final Proposal
## Luke Henry

For my final project, I plan to implement a semi-lagrangian fluid simulator to render smoke using volumetric rendering. The project will involve adding a SmokeField class that contains a 3D vector field of flow velocities and scalar fields for pressure, temperature, and smoke density. I will be basing the physics on the Incompressible Navier Stokes equation, and applying external buoyancy, gravity, and vortex confinement forces to correct for the inconsistencies and inacuracies of semi-lagrangian advection and the assumed constant densities. The project also involves rendering frames as the smoke fields change over small time steps. 

Semi-Lagrangian advection is a method for simulating fluid flow that uses a grid of flow velocities. It relies on the assumptions that all relevant fields follow the flow velocity field (are advected through it) and that the velocity at a given point does not change much over the course of a time step. Using the latter assumption, to find the new velocity at a point `x` where `t = n+1`, the process involves looking up the velocity `u(x,t=n)` and tracing it back in space to find the point `x - u(x,t=n)`. This new point can be thought of as "where the fluid at point `x` (time `t=n+1`) came from. Based on the assumption that this is a good enough estimator, as the time steps approach 0 seconds, the simulation then takes all of the field values from that point and sets them as the new field values at point `x` at time `t=n+1`. Once the new field values are set, the rest of the Navier Stokes Equation for Incompressible Fluids is applied as accelerations to each point in the new (time `t=n+1`) velocity field.

The Navier Stokes Equation for Incompressible Fluids with constant viscosity is:

![Nevier Stokes Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/e5e8521f648a2a1f7525f4f0dd166bbfbb079b0f)

(source: wikipedia)

Where `p` is pressure, `ρ` is density which can be set to 1 as it is constant and unitless, and `v` is viscosity which can also be constant and very low, such as 0.001 for gasses like air and smoke.

The main assumption for incompressible fluids is that the divergence of the velocity vield at any point is 0, so that density is constant. The main term in the equation is the Gradient of the pressure field, and in order for this assumption to hold, the following Poisson Equation for pressure must be solved to find the correct pressure field values:

![Poisson Pressure Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/9a0d9f8b11680878c6fe4cd016eb5e780ee1d980)

(source: wikipedia)

This process will involve solving an extremely large linear matrix equation `Ap = b` where A is a sparce matrix that has many 0's. From my understanding, the process of solving this requires using an iterative pressure solver, and will likely be the most difficult part of the simulator, but also the most important. 

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

I found this resource from Stanford describing a particular method for rendering smoke that has sources that link to the basics of fluid simulation as well, which I have been referencing with respect to the math.
[Fedkiw et al](chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://web.stanford.edu/class/cs237d/smoke.pdf)

I have the following gifs demonstrating (in order) advection with a constant velocity field, advection with a random velocity field, buoyancy and gravity forces on injected smoke:






