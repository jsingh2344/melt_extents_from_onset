# Generating Melt Extents and Snowlines from Onsets

## 11/6 - 11/13 update:


- Main issue was loading enough images from the datacube into memory at the same time to run functions on. Google Colab sadly was not fruitful. Currently chunking the datacube into 500x500 pixel sections, which equate to ~450 MB each.
  Some glaciers will be left out if they are on the boundary of chunks. Could clip directly to glacier bounding boxes and iterate through each glacier, or
  have the chunks overlap enough so that each glacier will be entirely contained in at least one chunk.
- Melt onsets are being triggered by a pixel's backscatter exceeding 3 dB from the winter mean. Melt extent is then calculated at any given
  time index by finding the number of pixels n that have onset up until that point in the current year, and then indexing into a flattened DEM with that n. Every pixel
  below that elevation is then deemed to be melting.
- Figs comparing this melt extent estimate to the previous workflow estimates are in me_cmp_figs. Each glacier, with above caveats, in path-frame 14-387 is included.
- A pixel triggers its 'snowline onset' by 1) having already begun its melt onset, and 2) by exceeding 4 dB from the 5th percentile summer backscatter.
  Will edit the 1) logic so that a pixel must only have been deemed 'melting'. Currently, some pixels at lower elevation anomalously never hit the 3 dB trigger,
  which incorrectly prevents them from ever having its 'snowline onset'.
- Wrote functions that output .tiff rasters for the snowline/melt extent DOY maps. 
- Other Todos: Replicate melt extent elevation logic for snowlines, write functions that convert purely from .tiffs to melt extent/snowline csvs, investigate post-processing methods,
  plot aggregate comparison diagnostics 


### 11/17:
- Fixed difference csv logic to include correct date range and plotted:
<img width="1389" height="690" alt="image" src="https://github.com/user-attachments/assets/07859745-b599-4bdd-9810-5ddfd9708986" />

- Old Melt Extent methodology comparison folder is at this commit link: https://github.com/jsingh2344/melt_extents_from_onset/tree/940c859bc244005947d52e1bffbd687a3e67ba82/me_cmp_figs

### 11/24:
- Investigating how, early in the summers, the onset method can yield a higher method than the allmelt method:
- First, a subset comparison plot for just summer months and some percentile bands:
- <img width="1389" height="690" alt="image" src="https://github.com/user-attachments/assets/8dc4c390-7d0f-4a15-870b-eaf3fc7e7a77" />
- Looking at glacier 6369, with and without the allmelt_threshold logic:
- <img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/272d52b1-2b2e-45db-b5a2-bb6ac4986bf8" /> is with, and
-  <img width="1200" height="600" alt="6369_no_allmelt_cmp" src="https://github.com/user-attachments/assets/09c1ce77-9141-4f87-9116-f2c5a487e0a2" /> is without
-  Going to examine in the second plot how the onset method gives such higher values. In a lot of cases, though, I'm guessing that early in the summer, the onset mask setting pixels
  to 'melted' once and then never reversing is why the onset method elevations are higher. Most of the allmelt method elevation graphs rise and then drop a few times, which could generate the delta.
- In particular, I will look at what is going on in 2020.
### 11/26 - 27:
- Looks like the difference is indeed that the onset logic has that pixels stay melting the whole year, and the previous allmelt methodology does not. Using persistent melt pixel logic (and no allmelt) for only 2020 gets:
- <img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/09140c4c-5ea5-4e6b-9daf-1fcd7aa02b3e" />
### 11/29:
- Extending to every year, to sanity check: <img width="1189" height="590" alt="f7ddc799-2c3e-4a78-9e5f-6cc72d8edf5c" src="https://github.com/user-attachments/assets/652e1b27-fd90-4939-8e8e-56c148441405" />
- Next, how it looks when using allmelt logic is added back in but the underlying pixel persistence is kept also (so the only difference between the onset method and the previous method should be because of added allmelt pixels):
- <img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/53563db8-c1fd-4c5f-9c3a-6fcb7e156dcf" />
- And here is the overall comparison, which matches expectation:
- <img width="1389" height="690" alt="image" src="https://github.com/user-attachments/assets/c75a9812-d151-43d6-a0fc-558ca7f01d30" />
- I believe the slight positive values in some glaciers is a result of using difference time indexing for the onset model (since it resets every year, I've been calculating elevations for 10-day intervals and using the nearest scene in time at each time index, which
  could mismatch the underlying data being used at a few comparison points)
- Also added the onset maps folder. Here are some examples for 6369:
  <img width="1199" height="1023" alt="6369_melt_onset_2018" src="https://github.com/user-attachments/assets/0ea955bc-1818-4149-acc3-85f8ec6ed34a" />
  <img width="1199" height="1023" alt="6369_melt_onset_2019" src="https://github.com/user-attachments/assets/a7ff28de-0021-4270-8c3e-eb2d4a6f6dc8" />
  <img width="1199" height="1023" alt="6369_melt_onset_2020" src="https://github.com/user-attachments/assets/66194267-b53f-40fb-9474-6f126216adad" />

## 12/12:

- Implemented logic for judging melt onset with pixels that show consecutive melt signal. In the onset_maps folder, glaciers 6360, 6369, 6023, 6025, 6026, 6029, 6031, 6043, 6047, 6352, 6353 (they share a grid chunk of the path frame)
  have figures for the updated onset maps, the old onset maps, the maps that difference the two. Other glacier folders have only the original onset maps at the moment. Examples for 6369:
  
<img width="1218" height="1023" alt="6369_melt_onset_2023" src="https://github.com/user-attachments/assets/4eb7b666-a638-4eff-8aae-1457487b62d5" />
<img width="1256" height="1023" alt="6369_melt_onset_cons_2023" src="https://github.com/user-attachments/assets/a4874507-8071-4859-82bc-6e4302999808" />
<img width="1301" height="1044" alt="6369_melt_onset_diff_2023" src="https://github.com/user-attachments/assets/0de8cd69-dfee-46ca-b079-5baaf368c7e6" />

- Single melt extent comparisons are in folder me_cmp_figs for the non-consecutive method, and me_cmp_figs_cons for the consecutive method
- Summary summer plot for the consecutive melt method is
  <img width="1389" height="690" alt="melt_extent_comparison_cons" src="https://github.com/user-attachments/assets/812f5e93-88d1-4ad9-9700-aaf06f7a20b0" />
- Added area proportion comparisons between the previous (non onset-based) method and onsets. Here they are for the non-consecutive onset method:
  <img width="1389" height="690" alt="area_comparison_no_cons" src="https://github.com/user-attachments/assets/878b05e1-22eb-4531-9bfe-0e1ec50973e2" />
  - And consecutive: 
  <img width="1389" height="690" alt="area_comparison_cons" src="https://github.com/user-attachments/assets/89488a6d-e136-43ce-a612-5f96ff63398b" />
- Finally, have some dotplots to compare methods also. Currently averaging elevation over each month for each glacier to get:
<img width="1781" height="2380" alt="monthly_prev_vs_onset_elev_may_oct" src="https://github.com/user-attachments/assets/4c4bb7bf-13b8-4caf-8a26-f2d40c0f6f98" />


, and


<img width="1780" height="2380" alt="monthly_prev_vs_onset_elev_may_oct_cons" src="https://github.com/user-attachments/assets/73a70f8b-79cd-473c-b4ec-1831a72b0ccb" />
(the second group of plots uses the consecutive melt logic)

And for area:

<img width="1784" height="2380" alt="monthly_prev_vs_onset_area_may_oct" src="https://github.com/user-attachments/assets/331d8782-53f6-4e6b-8481-519e8308d7d8" />

, and


<img width="1784" height="2380" alt="monthly_prev_vs_onset_area_may_oct_cons" src="https://github.com/user-attachments/assets/a355b6f7-a382-4c7e-8111-1af6e02bb144" />

Again, the second plot grouping uses consecutive melt logic. 



  












