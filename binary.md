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


```julia:plot_roche
#hideall

# unitless potential, mass in units of M1, distance in units of a
function a_div_ai(qf,qi)
    return (qi/qf)^(2)*((1+qi)/(1+qf))*exp(2*(qf-qi))
end

# check things by plotting a few equipotentials in the X-Y plane
using Plots

Mdonor = 30
Mfinal = 15


xvals = LinRange(15,30,100)
plot()
for Ma in [7,6,5,4]
    qvals = xvals./Ma
    qi = 30/Ma
    plot!(xvals, map(qf->log10(a_div_ai(qf,qi)), qvals), label=string(Ma)*"\$M_\\odot\$", legend=:bottomright)
end
xlabel!("\$M_\\mathrm{donor}\\;[M_{\\odot}]\$")
ylabel!("\$\\log(a/a_\\mathrm{i})\$")
savefig(joinpath(@OUTPUT, "shrinkage.svg"))

```
\figenv{Expected evolution of the orbital separation in terms of initial separation for a $30M_\odot$ donor star
being stripped by a companion through fully non-conservative mass transfer (assuming isotropic re-emission from the accretor). Each line is for a different accretor mass.}{/assets/lab2/shrinkage.svg}{width:100%;border: 1px solid gray;}

So if mass transfer can proceed stably for extreme mass ratios, we could potentially get an extreme reduction in orbital separation! This is particularly interesting in the context of gravitational wave sources \citep{vanDenHeuvel+2017}, where the time for two point masses to merge depends strongly on the orbital separation. For a circular orbit the merger time is \citep{Peters1964}:

$$t_\mathrm{merger}=\frac{a^4}{4B},\qquad B\equiv \frac{64G^3}{5c^5}(M_1+M_2)M_1M_2.$$

Taking arbitrarily high initial mass ratios could in principle lead to arbitrarily small post mass-transfer separations,
but there is a competition with mass transfer stability, which we have studied in the previous lab. The purpose of this lab
is to study whether or not at the boundary for stability the mass ratio is extreme enough to provide the required shrinkage in orbital
separation for gravitational waves to take over. We will consider a $30M_\odot$ donor star with different masses for a black hole companion, and compute a grid of simulations
using all the cores at disposition from the attending crowd.

## Setting up your template

You will need to make use of the template provided [here](../assets/lab2/part2.tar.gz).

## Modifying your `run_binary_extras`


## References

* \biblabel{
   Peters+1964}{Peters, P. C. (1964)} [Peters, P. C.](https://ui.adsabs.harvard.edu/abs/1964PhRv..136.1224P/abstract), Physical Review, vol. 136, Issue 4B, pp. 1224-1232, 1964
* \biblabel{Soberman+1997}{Soberman et al. (1997)} [Soberman, G. E.; Phinney, E. S.; van den Heuvel, E. P. J.](https://ui.adsabs.harvard.edu/abs/1997A%26A...327..620S/abstract), Astronomy and Astrophysics, v.327, p.620-635, 1997
* \biblabel{vanDenHeuvel+2017}{van den Heuvel et al. (2017)} [van den Heuvel, E. P. J.; Portegies Zwart, S. F.; de Mink, S. E.](https://ui.adsabs.harvard.edu/abs/2017MNRAS.471.4256V/abstract), Monthly Notices of the Royal Astronomical Society, Volume 471, Issue 4, p.4256-4264, 2017