

# ----- subtree creation --------------------------------------------------------------
git clone git@github.com:dwalbeck/job_track_now.git

# add nested repos
git subtree add --prefix job_track_now-api git@github.com:dwalbeck/job_track_now-api.git main --squash
git subtree add --prefix job_track_now-portal git@github.com:dwalbeck/job_track_now-portal.git main --squash


# ----- subtree pull ------------------------------------------------------------------
git subtree pull --prefix job_track_now-api git@github.com:dwalbeck/job_track_now-api.git main --squash
git subtree pull --prefix job_track_now-portal git@github.com:dwalbeck/job_track_now-portal.git main --squash


# ----- subtree push ------------------------------------------------------------------
git subtree push --prefix job_track_now-api git@github.com:dwalbeck/job_track_now-api.git main
git subtree push --prefix job_track_now-portal git@github.com:dwalbeck/job_track_now-portal.git main 

