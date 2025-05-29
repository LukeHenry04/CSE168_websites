# CSE 168 Final Proposal
## Luke Henry

For my final project, I plan to implement a semi-lagrangian fluid simulator to render smoke using volumetric rendering. The project will involve adding a SmokeField class that contains a 3D vector field of flow velocities and scalar fields for pressure, temperature, and smoke density. I will be basing the physics on the Incompressible Navier Stokes equation, and applying external buoyancy, gravity, and vortex confinement forces to correct for the inconsistencies and inacuracies of semi-lagrangian advection and the assumed constant densities. The project also involves rendering frames as the smoke fields change over small time steps. 

Semi-Lagrangian advection is a method for simulating fluid flow that uses a grid of flow velocities. It relies on the assumptions that all relevant fields follow the flow velocity field (are advected through it) and that the velocity at a given point does not change much over the course of a time step. Using the latter assumption, to find the new velocity at a point `x` where `t = n+1`, the process involves looking up the velocity `u(x,t=n)` and tracing it back in space to find the point `x - u(x,t=n)`. This new point can be thought of as "where the fluid at point `x` (time `t=n+1`) came from. Based on the assumption that this is a good enough estimator, as the time steps approach 0 seconds, the simulation then takes all of the field values from that point and sets them as the new field values at point `x` at time `t=n+1`. Once the new field values are set, the rest of the Navier Stokes Equation for Incompressible Fluids is applied as accelerations to each point in the new (time `t=n+1`) velocity field.

The Navier Stokes Equation for Incompressible Fluids with constant viscosity is:

![Nevier Stokes Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/e5e8521f648a2a1f7525f4f0dd166bbfbb079b0f)

(source: wikipedia)

The main assumption for incompressible fluids is that the divergence of the velocity vield at any point is 0, so that density is constant. The main term in the equation is the Gradient of the pressure field, and in order for this assumption to hold, the following Poisson Equation for pressure must be solved to find the correct pressure field values:

![Poisson Pressure Equation from Wikipedia](https://wikimedia.org/api/rest_v1/media/math/render/svg/9a0d9f8b11680878c6fe4cd016eb5e780ee1d980)

(source: wikipedia)


