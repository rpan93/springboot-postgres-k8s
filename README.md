# springboot-postgres-k8s
springboot-postgres-k8s

1. mvn clean install -DskipTests=true
2. docker build -t springboot-postgres-k8s:1.0 .
3. cd src/main/resources
4. kubectl apply -f postgres-credentials.yml
5. kubectl get secrets
6. kubectl apply -f postgres-configmap.yml
7. kubectl get configmaps
8. kubectl apply -f postgres-deployment.yml
9. kubectl get deployments
10. kubectl get services
11. kubectl get pods
12. kubectl apply -f deployment.yml
13. kubectl get deployments
14. kubectl get pods
15. kubectl logs -f <springboot-... pod>
16. kubectl exec -it <postgress... pod> bash
17. psql -h postgres -U postgres <password: postgres>
18. check database: \l
19. \c employeedb
20. \dt
21. select * from employee;

