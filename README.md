# 🚀 Argo CD ApplicationSet + Rollouts Repository

Repository นี้ใช้สำหรับบริหารจัดการ **Argo CD ApplicationSet** และ **Argo Rollouts** โดยแยกตาม project/service/environment อย่างเป็นระบบ  
รองรับ Deployment ทั้งแบบ **Blue-Green** และ **Canary** พร้อมโครงสร้าง Git ที่ออกแบบให้ทำงานร่วมกับ Helm และ GitOps ได้อย่างราบรื่น

---

## 📁 โครงสร้างของ Repository

```bash
argocd-devops/
├── argo-cd/                             # 🎯 สำหรับ Argo CD ApplicationSet
│   └── applicationset/
│       └── ironman/
│           └── applicationset-git-generators/
│               ├── applicationset-git-generators-bluegreen.yaml   # 🎨 BlueGreen ApplicationSet
│               └── applicationset-git-generators-canary.yaml      # 🐦 Canary ApplicationSet

├── argo-rollout/                        # 🚦 สำหรับ Rollout strategy โดยเฉพาะ
│   └── applicationset/
│       └── ironman/
│           └── applicationset-git-generators/
│               ├── rollout-bluegreen.yaml                         # 🔵 Rollout สำหรับ BlueGreen
│               └── rollout-canary.yaml                            # 🐤 Rollout สำหรับ Canary

└── README.md                            # 📘 ไฟล์นี้แหละ!

---
🔧 สิ่งที่คุณต้องมี
✅ ติดตั้ง Argo CD + Argo Rollouts แล้วบน Kubernetes Cluster
---
✅ Repository helm-chart/ และ argocd-demo/ ที่แยกเก็บค่า Helm values และ app structure
---
✅ Helm chart ที่รองรับ rollout strategy (อยู่ใน repo helm-chart/)
---
✅ วิธีใช้งาน

1. Apply ApplicationSet สำหรับ Blue-Green
bash
คัดลอก
แก้ไข

kubectl apply -f argo-cd/applicationset/ironman/applicationset-git-generators/applicationset-git-generators-bluegreen.yaml

2. Apply ApplicationSet สำหรับ Canary

kubectl apply -f argo-cd/applicationset/ironman/applicationset-git-generators/applicationset-git-generators-canary.yaml

3. ตรวจสอบ Application ที่สร้างบน Argo CD UI
เปิด Argo CD UI

ดู Application ที่ถูก generate จาก ApplicationSet

สามารถ Sync, Promote, Rollback ได้จากหน้า UI

🙌 Contributors
Created by noppon chantaban