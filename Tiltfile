# -*- mode: Python -*-

docker_build(
  'rollout-demo',
  '.',
  dockerfile='./src/main/docker/Dockerfile',
  live_update=[sync('./src/', '/project/src'),
               sync('./pom.xml', '/project/pom.xml') ],
  entrypoint = 'mvn quarkus:dev')

k8s_yaml('dev-env/app.yaml')
k8s_resource('rollout-demo', port_forwards=[8080,5005])