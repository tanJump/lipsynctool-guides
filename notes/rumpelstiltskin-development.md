# Implementing the Rumpelstiltskin two-person video: from Kling to H3

When I built this feature for LipSyncTool, the goal was simple: upload two photos, one to replace the dancer and one to replace the bystander, while keeping the template’s motion, scene, and music.

Testing kept hitting the same problem, though: the API returned success, the video played, but the people weren’t swapped correctly.

## Kling: could swap one person, but swapping two at once was unreliable

I first used an 8-second barn template I’d generated myself. Kling O3 Edit swapped the male dancer; the female bystander barely changed. Later, when I swapped only the woman, her face and hair changed, but the black neckline from the reference photo got carried into the original dress.

I switched to AIGenTools’ 6-second template and tested again. Same result: the man changed noticeably, but neither of the woman’s close-ups met the swap requirement. The soundtrack was preserved pretty well, but the cut timing drifted.

I checked the actual requests: both photos were uploaded, the asset URLs were reachable, and I saved the video the vendor returned. Nothing was missing or saved to the wrong file.

I never pinned down the exact cause. This API mainly relies on the prompt to map reference photos to people in the video—there’s no explicit target-person ID or mask. The woman’s close-ups are only about 0.6 seconds each, and she’s covering her face with her hand, which may have made it harder. But the earlier fixed-camera template failed too, so I can’t just blame the cuts.

Also, there was only one reference photo per person. Adding more reference images just repeated the same image and didn’t add any new angles. That’s a weakness in the input, but I don’t have a controlled test proving it was the root cause.

## Seedance: one attempt failed on quality, two failed asset moderation

Seedance 2.0 Fast generated a video from a different 10-second template, but the man’s appearance wasn’t consistent across shots, and the woman still had the template’s hairstyle. Because the assets were different, I can’t directly compare it with Kling based on this.

Seedance 2.5 Editing used the same 6-second template and photos as Kling. Both attempts were rejected during asset moderation, so no video was generated. The error mentioned real-person likeness or private information, but didn’t say which asset was the problem.

Those two attempts only show that the API at the time wouldn’t accept this set of inputs. They don’t tell us anything about generation quality.

## H3: both people finally changed, but the shots didn’t survive

H3 got closer to what I wanted. In several shots, the man had the glasses, beard, and hairstyle from the reference photo, and the woman’s final close-up changed clearly too.

But the template changed: seven shots became six, the first close-up of the woman reacting disappeared, the cut rhythm changed, and the output went from about 6 seconds to about 6.58 seconds.

The API call took about 111 seconds, and the estimated cost from the response was about $0.72. I didn’t separately check it against the bill. I also adjusted the prompt, so this was just a same-asset comparison of approaches—it doesn’t prove H3 is consistently better than Kling.

H3 didn’t preserve the original music. I later merged the template’s audio track back into the video separately. The audio can match, but the video is longer, with about half a second of silence at the end. Merging the audio can’t fix the changed cuts or the timing.

## Final trade-off

I ended up using H3, then merging the template soundtrack back in after generating the video. I stopped promising “only the faces change, everything else stays exactly the same.” Now I explicitly warn that likeness, clothing, shots, and pacing may all change.

I also looked into options with explicit character mapping, like AKOOL, but didn’t test them and haven’t integrated them for now. Doing two sequential swaps was another candidate, but I never got it working.

This round changed how I judge things: a vendor finishing the job just means you got a file. Whether both people were actually swapped, whether a shot got dropped, whether the music lines up—those are separate checks.

Right now I only have a small set of test results, not enough to guarantee that any two photos will work. The next step should be testing multiple pairs of people on the same template, especially the woman’s two short close-ups—not writing an even longer prompt.
