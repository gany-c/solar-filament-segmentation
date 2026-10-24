# Solar Astronomy & Filament Segmentation Glossary

A reference guide for solar physics, observatories, instrumentation, and optical concepts relevant to the MAGFiLO solar filament segmentation dataset.

---

## 1. Solar Phenomena & Features

* **Solar Filament:** A cool, dense, thread-like channel of plasma (hydrogen gas) suspended above the hot solar chromosphere and corona by magnetic fields. When viewed against the bright solar disk, filaments appear as dark absorption features.
* **Prominence:** The exact same physical structure as a solar filament, but observed projecting off the edge (limb) of the solar disk against the dark background of space, appearing as a bright, glowing loop of plasma.
* **Active Region Filament:** A smaller, dynamic filament formed in or near active solar regions and sunspot groups. These follow the standard ~11-year Sunspot Cycle (occurring at lower equatorial solar latitudes).
* **Polar Crown Filament:** A long, sprawling, highly elongated filament that forms at high solar latitudes along the border separating the unipolar magnetic field near the solar poles. They follow a distinct cycle ("rush to the poles") slightly offset from the equatorial sunspot cycle.
* **Polarity Inversion Line (PIL):** The neutral boundary line separating regions of opposite magnetic polarity (north/positive vs. south/negative) on the solar surface. Solar filaments physically condense and form directly above PILs.
* **Coronal Mass Ejection (CME):** A massive eruption of solar magnetic fields and associated plasma thrown into space. Filament eruptions can accompany CMEs, which can cause geomagnetic storms if directed toward Earth (Geo-effective).
* **Chromosphere:** The thin layer of the Sun's atmosphere located directly above the photosphere and below the corona, where chromospheric phenomena like filaments, fibrils, and spicules are observed.
* **Sunspots & Active Regions:** Temporary dark spots on the photosphere caused by intense magnetic field concentrations that inhibit convection.

---

## 2. Observatories, Instrumentation & Datasets

* **Spectral Line:** A dark or bright line in an optical spectrum resulting from an excess or deficiency of photons in a narrow frequency/wavelength range compared with nearby frequencies. In solar astronomy, an **absorption line** (like the H-alpha line at $656.28\text{ nm}$) occurs when cooler plasma—such as a solar filament suspended above the disk—absorbs specific light wavelengths emitted by the hotter layers beneath it, creating a dark, recognizable feature against the brighter solar surface.
* **H-alpha ($\mathbf{H\alpha}$) Filter / Observation:** A deep-red narrow spectral line of hydrogen ($656.28\text{ nm}$) commonly used in solar astronomy. H-alpha filters isolate light from the solar chromosphere, making filaments, prominences, and flares sharply visible.
* **GONG Network (Global Oscillation Network Group):** A worldwide network of six identical, fully automated solar observing stations designed to monitor the Sun as continuously as possible. The six stations are:
  * **Bh:** Big Bear Solar Observatory (California, USA)
  * **Ch:** Cerro Tololo Inter-American Observatory (Chile)
  * **Lh:** Learmonth Solar Observatory (Australia)
  * **Mh:** Mauna Loa Solar Observatory (Hawaii, USA)
  * **Th:** Teide Observatory (Tenerife, Canary Islands, Spain)
  * **Uh:** Udaipur Solar Observatory (Rajasthan, India)
* **MAGFiLO Dataset:** The benchmark solar filament dataset comprising $2048 \times 2048$ pixel H-alpha solar images collected across the GONG network, annotated with COCO segmentation masks.
* **CrOTA:** This instrument name and its association with an observatory need verification against the original reference before use.
* **AIA (Atmospheric Imaging Assembly):** An imaging instrument aboard NASA's Solar Dynamics Observatory (SDO) capturing multi-wavelength UV and EUV (e.g., $304\text{ \AA}$) images of the solar atmosphere.
* **SST / BBSO:** High-resolution solar observatories (Swedish 1-m Solar Telescope and Big Bear Solar Observatory) that reveal individual thread-like fine structures within filament spines.

---

## 3. Optics, Image Processing & Artifacts

* **Limb Darkening:** An optical effect seen in solar images where the center of the solar disk appears brighter than the outer edge (limb) because the line of sight penetrates deeper into hotter atmospheric layers at the center.
* **Limb Darkening Correction:** Image preprocessing algorithms applied to solar disk images to flatten the background intensity from center to edge.
* **Full Width at Half Maximum (FWHM):** A measure of the bandpass width of narrow optical filters (such as H-alpha solar filters) or point-spread functions in solar imagery.
* **Quiet-Sun vs. Active-Sun:** Terminology describing periods or regions of low solar magnetic activity (few/no sunspots/filaments) versus high activity (numerous active regions, flares, and filaments).
* **Hardware Artifacts / Outer Rings:** Visible borders or rings that may reflect optics, image processing, or other acquisition effects. A sample image alone does not establish the cause or a station-wide pattern.

## 4. Terms for interpreting the analysis

* **Unique image vs. annotation record:** Multiple annotators may label the same photograph. Different JSON image IDs do not necessarily mean different photographs.
* **Instance mask:** A mask for one filament. A union of masks is useful for display but loses individual object identities.
* **Elongation:** A shape descriptor such as a fitted ellipse axis ratio. It does not measure physical length or identify a solar filament type.
* **Validation leakage:** Related observations appearing in both training and validation can inflate measured performance. Group repeated filenames together and discuss temporal grouping before selecting a split.
* **Projected position vs. solar latitude:** An image Y-coordinate alone is not a physical latitude. Geometry and viewing orientation must also be considered.

## References and scope

- GONG stations and observing network: https://nso.edu/telescopes/nisp/gong/
- Dataset and annotation format: https://www.kaggle.com/competitions/filament-segmentation-2026/data
- Task and permitted inputs: https://www.kaggle.com/competitions/filament-segmentation-2026

Definitions are introductory. Image artifacts and proposed modeling consequences remain hypotheses until checked against representative data and validation results.
