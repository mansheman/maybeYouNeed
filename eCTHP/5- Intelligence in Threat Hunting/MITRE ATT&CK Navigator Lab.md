2026-04-13 00:00

Status:

Tags: [[eCTHP]] [[Threat Intelligence]]
###### Prerequisites: [[Kill Chain and ATT&CK]]
# MITRE ATT&CK Navigator Lab

## Goal

Use MITRE ATT&CK Navigator to:

- map observed TTPs
- compare them against known threat groups
- identify overlap and coverage gaps

## Tasks

- Task 1: Map Observed TTPs
- Task 2: Add Threat Actor Layers
- Task 3: Compare and Analyze Overlap

## Step 1

Access the Ubuntu machine.

![[Pics/ecthp/mitre-lab/mitre-lab-01.png|1200]]

## Step 2

Open Firefox and browse to:

```text
http://localhost:4200
```

![[Pics/ecthp/mitre-lab/mitre-lab-02.png|1200]]

## Task 1: Map Observed TTPs

### Step 3

Create a new layer for the Enterprise matrix.

![[Pics/ecthp/mitre-lab/mitre-lab-03.png|1200]]

![[Pics/ecthp/mitre-lab/mitre-lab-04.png|1200]]

### Step 4

Go to `Layer Controls > Settings` and rename the layer to `MyTTPs`.

![[Pics/ecthp/mitre-lab/mitre-lab-05.png|1200]]

### Step 5

Because the observed TTPs relate to Windows, configure the platform as `Windows`.

Go to `Layer Controls > Filters` and deselect every option except `Windows`.

![[Pics/ecthp/mitre-lab/mitre-lab-06.png|1200]]

Then go to `Selection Controls > Selection Behavior` and disable `Select techniques across tactics` so the same technique is not plotted under multiple tactics.

![[Pics/ecthp/mitre-lab/mitre-lab-07.png|1200]]

### Step 6

Plot the observed TTPs by selecting them one by one while holding `Shift`.

![[Pics/ecthp/mitre-lab/mitre-lab-08.png|1200]]

After selecting the TTPs, go to `Layer Controls > Colour Setup`. Under `Scoring Gradient`, add the color `#43c8bb`, then set:

- `Low value` to `1`
- `High value` to `4`

![[Pics/ecthp/mitre-lab/mitre-lab-09.png|1200]]

Next, go to `Technique Controls > Scoring` and assign a score of `1`.

![[Pics/ecthp/mitre-lab/mitre-lab-10.png|1200]]

The selected techniques should now be highlighted in red.

## Task 2: Add Threat Actor Layers

### Step 7

Open a new tab in Navigator and create a new Enterprise layer. Rename it to `APT28` and set the platform to `Windows`.

![[Pics/ecthp/mitre-lab/mitre-lab-11.png|1200]]

### Step 8

Go to `Selection Controls > Search & Multiselect`. Search for `APT28`, then under `Threat Groups` select `APT28`.

![[Pics/ecthp/mitre-lab/mitre-lab-12.png|1200]]

This selects the techniques associated with APT28.

### Step 9

Go to `Layer Controls > Colour Setup` and apply the same scoring gradient used for the `MyTTPs` layer.

![[Pics/ecthp/mitre-lab/mitre-lab-13.png|1200]]

### Step 10

Go to `Technique Controls > Scoring` and assign a score of `2`.

![[Pics/ecthp/mitre-lab/mitre-lab-14.png|1200]]

The selected techniques should now appear highlighted in yellow.

### Step 11

Repeat steps 7 through 9 for `APT41`, then assign a score of `3`.

![[Pics/ecthp/mitre-lab/mitre-lab-15.png|1200]]

The APT41 techniques should now appear highlighted in green.

## Task 3: Compare and Analyze Overlap

### Step 12

Open a new tab in Navigator and expand `Create Layers from Other Layers`. Set the domain to `Enterprise ATT&CK v17.1`.

To compare `MyTTPs` with `APT28`, use the score expression:

```text
a + b
```

Here:

- `a` is the `MyTTPs` layer
- `b` is the `APT28` layer

Set the gradient to `MyTTPs` so the scoring gradient is reused.

![[Pics/ecthp/mitre-lab/mitre-lab-16.png|1200]]

Click `Create Layer`.

![[Pics/ecthp/mitre-lab/mitre-lab-17.png|1200]]

### Step 13

Set the platform to `Windows`, then under `Scoring Gradient` change the third color to `#922995`. This highlights overlapping techniques in purple. The overlap score here is `3`.

![[Pics/ecthp/mitre-lab/mitre-lab-18.png|1200]]

This reveals where the observed activity overlaps with APT28.

### Step 14

To compare `MyTTPs` with `APT41`, repeat the same process in a new tab with:

```text
a + c
```

Here:

- `a` is the `MyTTPs` layer
- `c` is the `APT41` layer

Click `Create Layer`.

![[Pics/ecthp/mitre-lab/mitre-lab-19.png|1200]]

Set the platform to `Windows`, then change the fourth color in the scoring gradient to `#922995`. This highlights overlapping techniques in purple. The overlap score here is `4`.

![[Pics/ecthp/mitre-lab/mitre-lab-20.png|1200]]

## Analysis

The overlap with APT41 is greater than the overlap with APT28, which makes APT41 the stronger fit for the observed behavior.

The exercise also exposes detection gaps in:

- Privilege Escalation
- Credential Access
- Discovery

These should be prioritized for better detection coverage.

## Conclusion

This lab shows how ATT&CK Navigator can be used to:

- visualize observed TTPs
- compare them against known adversary tradecraft
- support rough attribution
- identify defensive blind spots

## Reference

- [MITRE ATT&CK Navigator](https://github.com/mitre-attack/attack-navigator)
