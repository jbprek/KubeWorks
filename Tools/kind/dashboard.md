# 1 Create Service Account
````bash
cat << 'EOF' > service-account.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

kubectl apply -f service-account.yml
````

# 2 Get the token for the account
````bash
kubectl describe serviceaccount admin-user -n kubernetes-dashboard
````

Copy the value from the above output fotr key Tokens:

````bash
kubectl describe secret <TOKENS_VALUE> -n kubernetes-dashboard

# Example
kubectl describe secret admin-user-token-s7v2n -n kubernetes-dashboard
````
##Example Output Token
eyJhbGciOiJSUzI1NiIsImtpZCI6IlpnSGRzQ1RCSGJfWXJ1ZkJ3WmU5VXg1N3lNbm5VRTR6WHk5QkhXalNDZWsifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXM3djJuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyZDZjNTllOS00NDU3LTQ4MzMtOThjMi0yODZmYWUxOTRmMjgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.P8YOzT9D8gnEdgmsn4BRgBKmth3NN2tFCXyT62Sxu-tSR1oeY6eHSW4YH6y9HKmZz2ApRLrkJxOKGh0BEBaH7HXBS6pbAToCn07xB4B8Wm2qhxgQdEI-mnn59FjO_JoBh409JXhp-1d6mz6OS7FaCTGlsH4ubIEnOF3h0CIre7-6WO3lyQf6ivBzpTFNJRFNwOYAfclqP5buqIuYcCcsmavQgBlw9YW1aNiOYiD_ebDVN38hNf8VR10QeWJIi5kS9I0D3VptNqF1iSxiNgN-HwBHXIFrQEESvGRrG0JuibrCTqqfC5ecUuTrT_adbTHJGRLobCVCaC-1Njou_Atshw

Use the above to login:
[Kubernetes Login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:dashboard-kubernetes-dashboard:https/proxy/#/login)