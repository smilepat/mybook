# 작업 마무리 체크리스트

배포 또는 작업 완료 시 아래 항목을 확인할 것:

- 새로운 환경변수가 추가되었다면 배포 플랫폼(Vercel 등)에 모든 환경(production, preview, development)에 등록할 것
- `.env.example` 파일도 함께 업데이트할 것
- DB 스키마나 시드 데이터가 변경되었다면 원격 DB에도 반영할 것
- 다른 PC에서 clone 후 바로 작업 가능한 상태인지 확인할 것

> 이렇게 하면 Turso, Supabase, PlanetScale 등 어떤 DB든, Vercel, Netlify 등 어떤 플랫폼이든 적용됩니다.
