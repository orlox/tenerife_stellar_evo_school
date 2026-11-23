+++
title = "Binary simulations"
hascode = true
rss = ""
rss_title = "Binary simulations"
rss_pubdate = Date(2026, 11, 23)

tags = ["binary", "mesa"]
+++

# Binary simulations

\toc

Under the assumption that mass transfer is extremely inefficient (all mass ejected) and lost material
takes away the specific orbital angular momentum of the accretor, the evolution of
the orbital separation $a_\mathrm{f}$ can be analitically described. In terms of the final mass ratio $q_\mathrm{f}\equiv m_\mathrm{donor,f}/m_\mathrm{accretor,f}$
and a separation $a_{i}$ at which the system had a mass ratio $q_\mathrm{i}$ we have \citep{soberman+1997}:

$$\frac{a_\mathrm{f}}{a_\mathrm{i}}=\left(\frac{q_\mathrm{i}}{q_\mathrm{f}}\right)^2\left(\frac{1+q_\mathrm{i}}{1+q_\mathrm{f}}\right)\exp\left(2[q_\mathrm{f}-q_\mathrm{i}]\right).$$

The initial and final points needs not refer to that at the onset or end of the mass transfer phase, they can also be any arbitrary point during the process. From that we can estimate how much the orbital separation would change respect to the initial one in terms of the accretor mass and the mass of the companion. In particular, the figure below shows an example of a $30M_\odot$ star being stripped down to $15M_\odot$ by companions of different masses. 

\figenv{Expected evolution of the orbital separation in terms of initial separation for a $30M_\odot$ donor star
being stripped by a companion through fully non-conservative mass transfer (assuming isotropic re-emission from the accretor). Each line is for a different accretor mass.}{/assets/lab2/shrinkage.svg}{width:100%;border: 1px solid gray;}

So if mass transfer can proceed stably for extreme mass ratios, we could potentially get an extreme reduction in orbital separation! This is particularly interesting in the context of gravitational wave sources \citep{vanDenHeuvel+2017}, where the time for two point masses to merge depends strongly on the orbital separation. For a circular orbit the merger time is \citep{Peters1964}:

$$t_\mathrm{merger}=\frac{a^4}{4B},\qquad B\equiv \frac{64G^3}{5c^5}(M_1+M_2)M_1M_2.$$

Taking arbitrarily high initial mass ratios could in principle lead to arbitrarily small post mass-transfer separations,
but there is a competition with mass transfer stability, which we have studied in the previous lab. The purpose of this lab
is to study whether or not at the boundary for stability the mass ratio is extreme enough to provide the required shrinkage in orbital
separation for gravitational waves to take over. We will consider a $30M_\odot$ donor star with different masses for a black hole companion, and compute a grid of simulations
using all the cores at disposition from the attending crowd.

Owing to time constraints we will not model the full mass transfer phase, but only run simulations until the $30M_\odot$ donor is stripped to $20M_\odot$. From this point we will assume mass transfer will be stable until it strips the star down to its helium core mass, and estimate from that if we can produce a merger due to GW emission in the lifetime of the Universe. We will also focus on case B simulations, as case A simulations can be significantly more expensive to run, and there is no well defined helium core mass to make the approximation stated before.

## Setting up your template

You will need to make use of the template provided [here](../assets/lab2/part2.tar.gz). This simulation models only one component, while the black hole is taken to be a point mass with mass accretion limited to its Eddington rate. Just as with the single star template, you can compile this one by doing:
```bash
./clean && ./mk
```
and then run it with
```
./rn | tee out.txt
```
Beware that we will be modifying the source files under `src`, so we will need to recompile the directory every time we do so. It is not necessary to recompile if one only modifies the inlist files.To illustrate a bit adjustments to inlist options, we will add a stopping condition once the mass of the donor goes below $20M_\odot$, or if it reaches core helium depletion. This will let us focus on systems undergoing case B mass transfer, systems hitting the helium core depletion would not have undergone meaningful stripping. To do this, add the following anywhere in the `&controls` section of your inlist:

```fortran
   !Stop when a large fraction of the star has been stripped
   star_mass_min_limit = 20d0
   !stop when the center mass fraction of he4 drops below this limit
   xa_central_lower_limit_species(1) = 'he4'
   xa_central_lower_limit(1) = 1d-3
```

The initial conditions that we will modify are the black hole mass and the initial orbital period. These can be adjusted in `inlist_project` by changing the relevant parameters
```fortran
&binary_controls
   ...
   m1 = 30.0d0 ! donor mass in Msun
   m2 = 8d0 ! companion mass in Msun
   initial_period_in_days = 10d0
   ...
/ ! end of binary_controls namelist
```

## Modifying your `run_binary_extras`

MESA does not really have a built-in concept of unstable mass transfer, so we need to give it one. We will add two new terminating conditions in `src/run_binary_extras.f90` which are not provided as inlist options. Within these file there are multiple subroutines that allows us to add code in different stages of the simulation. In particular, we can modify the function `extras_binary_finish_step` (which is run after each step is completed) to do this:

```fortran
! returns either keep_going or terminate.
! note: cannot request retry; extras_check_model can do that.
integer function extras_binary_finish_step(binary_id)
   type (binary_info), pointer :: b
   integer, intent(in) :: binary_id
   integer :: ierr
   real(dp) :: mdot_th, mdot_dyn, avg_rho
   call binary_ptr(binary_id, b, ierr)
   if (ierr /= 0) then ! failure in  binary_ptr
      return
   end if
   extras_binary_finish_step = keep_going

   mdot_th = b% m(1)/(standard_cgrav*(b% m(1))**2/(b% s1% L(1)*b% r(1)))

   ! finish if the mass transfer rate is significantly above the thermal rate
   write(*,*) "check mdots", mdot_th/Msun*secyer
   if (abs(b% mtransfer_rate)>100d0*mdot_th) then
       write(*,*) "Finish simulation due to high mass transfer rate"
       extras_binary_finish_step = terminate
   end if

  ! finish if the binary separation does not make sense anymore
   if (b% r(1) > b% separation) then
       write(*,*) "Finish simulation due to radius exceeding separation"
       extras_binary_finish_step = terminate
   end if

end function extras_binary_finish_step
```
We can also incorporate the calculation of the merger time as discussed above. For this we can make use of the function `extras_binary_after_evolve`, to prform a calculation at the very end of any simulation and print it out in the terminal:
```fortran
subroutine extras_binary_after_evolve(binary_id, ierr)
   type (binary_info), pointer :: b
   integer, intent(in) :: binary_id
   integer, intent(out) :: ierr
   real(dp) :: m1f, m2f, qi, qf, ai, af, Bmerge, tmerge
   call binary_ptr(binary_id, b, ierr)
   if (ierr /= 0) then ! failure in  binary_ptr
      return
   end if

   qi = b% m(1)/b% m(2)

   m1f = b% s1% he_core_mass*Msun ! assume stripping down to the helium core
   m2f = b% m(2) ! assume no further accretion
   qf = (b% s1% he_core_mass*Msun)/b% m(2)

   ai = b% separation
   af = ai*(qi/qf)**2*((1+qi)/(1+qf))*exp(2*(qf-qi))

   Bmerge = 64d0/5d0*standard_cgrav**3/clight**5*(m1f+m2f)*m1f*m2f
   tmerge = af**4/(4d0*Bmerge)

   write(*,*) "Merger time in Gigayears", tmerge/secyer/1e9

end subroutine extras_binary_after_evolve
```
Go through this code and make sure you understand its purpose. In doubt summon your nearest MESA expert.

**BE SURE TO RECOMPILE YOUR WORK FOLDER AFTER MODIFYING THE SOURCE CODE FILES!**

## Fill-up the grid!
Now, with everything ready, go to [this spreadsheet](https://docs.google.com/spreadsheets/d/1kVxDli11I1FJF5rGQo347_j8CNiVP6uInDz4Jqmo7A8/edit?usp=sharing) and start claiming initial parameters, doing runs, and filling this grid. As it starts to populate, select systems carefully to try and identify the region where GW mergers would happen!

## References

* \biblabel{
   Peters+1964}{Peters, P. C. (1964)} [Peters, P. C.](https://ui.adsabs.harvard.edu/abs/1964PhRv..136.1224P/abstract), Physical Review, vol. 136, Issue 4B, pp. 1224-1232, 1964
* \biblabel{Soberman+1997}{Soberman et al. (1997)} [Soberman, G. E.; Phinney, E. S.; van den Heuvel, E. P. J.](https://ui.adsabs.harvard.edu/abs/1997A%26A...327..620S/abstract), Astronomy and Astrophysics, v.327, p.620-635, 1997
* \biblabel{vanDenHeuvel+2017}{van den Heuvel et al. (2017)} [van den Heuvel, E. P. J.; Portegies Zwart, S. F.; de Mink, S. E.](https://ui.adsabs.harvard.edu/abs/2017MNRAS.471.4256V/abstract), Monthly Notices of the Royal Astronomical Society, Volume 471, Issue 4, p.4256-4264, 2017