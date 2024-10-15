# Terry's lab template

## Cloning a new lab from this template
- Create a new repo in github using UI/command as usual. This will be the target repo where duplicate of source repo to be pasted.
- Set default branch as gh-pages:  (>settings>branches>switch>gh-pages)
- From a Linux terminal:
    - git clone --bare https://github.com/koziniec/lab-template
    - cd lab-template.git
    - git push --mirror https://targetRepoURL

## Preparing the new lab
- Update values in /_config.yml
- put all images in: /assets/img
    - /assets/img/topology.png
- update copyright licence information in: /pages/about.md 
- Update lab title(s) here: /pages/index.md    (Don't change topology link)
- Put EVE topology files here: /_docs/eve   (initial.zip and final.zip)
- Build lab content here: /_docs/lab-exercise.md
