FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=a9ff811498b5b71667ad199766a0e746ab37d508
RUN git clone https://github.com/transmute-app/transmute . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# transmute's backend imports python-magic and cairosvg at module import time (backend/core, conftest);
# upstream docker/Dockerfile installs these runtime libraries. Heavy converters (ffmpeg, ghostscript,
# inkscape, tesseract, libreoffice, calibre) are left out: CI skips the end-to-end conversion tests.
RUN apt-get update \
    && apt-get install -y --no-install-recommends libmagic1 libsm6 libxext6 libopengl0 libosmesa6 \
       libpango-1.0-0 libpangocairo-1.0-0 libgdk-pixbuf-2.0-0 libcairo2 librsvg2-bin libffi-dev \
    && rm -rf /var/lib/apt/lists/*
RUN pip install --no-cache-dir \
    -r requirements.txt \
    && pip install --no-cache-dir pytest \
    && test -z "$(git status --porcelain)"

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
