# using buildah 


buildah1 bud -f Dockerfile  -t hello:docker --format docker


buildah1 push localhost/hello:docker docker-daemon:hello:1



