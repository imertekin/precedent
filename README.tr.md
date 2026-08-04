# Precedent

**Precedent, Project Historian rolünü implemente eder** — AI-destekli yazılım projelerinde çoğunlukla eksik olan bir görev: kararları koruyan, projeyi hatırlayan, geçmişini yöneten, bayatlayanları gözden geçiren ve kendi metodolojisini sürdüren bir ajan — bunu, kodu gerçekten yazan ajandan ayrı tutarak.

Somut olarak: bir planlama ajanı (örn. Cowork'taki Claude) ve bir kod ajanı (örn. Claude Code) küçük bir dosya setini paylaşır, böylece kararlar kaybolmadan koda akar — ve "bunu neden böyle yapmıştık?" sorusu her seferinde yeniden tartışılmak zorunda kalmaz.

İsim hukuki bir metafor: "precedent" (emsal/içtihat), geçmiş bir kararın gelecekteki kararları, davayı sıfırdan tekrar tartışmaya gerek kalmadan bağlaması ve yönlendirmesi anlamına geliyor. Bir karar günlüğünün işlevi de tam olarak bu.

Bu repo iki şeyi bir arada verir: bu doküman, ve olduğu gibi kurulabilen çalışan bir `skill/` — Project Historian rolünün bir implementasyonu, tek olası implementasyonu değil. Bu README'nin İngilizce hali [`README.md`](./README.md) dosyasında.

## Önce bir uyarı

Buradaki fikirlerin neredeyse hiçbiri yeni değil. Karar günlükleri, spec-önce yaklaşımı, bağlam ekonomisi — hepsi yazılım mühendisliğinde yerleşik, isimlendirilmiş, bazıları resmi standart haline gelmiş pratikler. Bu repo onları icat ettiğini iddia etmiyor. Amaç, bu bilinen parçaların iki-ajanlı (planlayıcı + uygulayıcı) bir akışta nasıl bir araya geldiğini dürüstçe göstermek — "biz keşfettik" değil "böyle birleştirdik" çerçevesinde.

Bu aynı zamanda, birimizin gerçekten günlük kullandığı, Türkçe ve şirkete özel bir skill'in genelleştirilmiş, İngilizce halidir. Çalışan versiyonda sabit kategoriler ve sabit bir çalışma dili var — bunlar sadece o özel kuruluma anlamlı; bu versiyon onları çıkarıp her yerde şablon olarak kullanılabilir hale getiriyor.

## Problem

AI ajanlarıyla oturumlar boyunca çalışmak iki farklı bağlam kaybı üretir:

1. **Oturum kaybı** — her yeni konuşmada projeyi sıfırdan anlatmak.
2. **Karar kaybı** — bir kararın neden alındığını, aylar sonra kimsenin hatırlamaması.

Buna iki geniş yaklaşım var: otomatik hafıza sistemleri (embedding tabanlı retrieval, arka planda çalışan sunucular) ve elle küratörlüğü yapılmış, insan tarafından düzenlenen dokümanlar. Precedent ikincisini seçiyor — ve bu seçim ölçeğe bağlı: az sayıda projesi, seyrek ve bilinçli kararları olan bir kurulumda (küçük bir stüdyo, birkaç ürün) manuel küratörlük hem daha ucuz hem daha güvenilir. Yüksek hacimli, çok oturumlu bir ortamda otomatik bir hafıza sistemi muhtemelen daha uygun olurdu. Bu metodoloji o senaryo için tasarlanmadı.

## Precedent, Project Historian rolünü implemente eder

Burada gerçekte farklı olanı anlatmanın en net yolu şu: planlama ajanı sadece plan yapmıyor, bir role bürünüyor — proje tarihçisi ve küratörü — bu, iyi işleyen mühendislik ekiplerinde zaten var olan ama çoğu güncel AI kodlama kurulumunda net tanımlanmamış bir rol, çünkü çoğu kurulum planlama/uygulama/hafızayı tek, farklılaşmamış bir ajanda topluyor. Somut olarak bu rolün beş görevi var, her biri bu repodaki belirli bir mekanizmaya karşılık geliyor:

- **Kararları korur.** `docs/DECISIONS.md`, ADR formatında, düz bir "alındı/alınmadı" yerine bir yaşam döngüsüyle (Proposed → Accepted → Implemented → Validated, ya da Rejected / Deprecated / Superseded).
- **Projeyi hatırlar.** `AGENTS.md`, her oturum başında yeniden yüklenir, sıfırdan anlatmak yerine bağlamı ekonomik tutar.
- **Geçmişi yönetir.** `DECISIONS_DRAFT.md` inbox'ı, hepsi aynı şekilde temizlenmek yerine Accepted / Rejected / Ignored olarak triaj edilir — reddedilen bir not bile kaybolmadan iz bırakır.
- **Review yapar.** Session Start Checklist, her ADR'nin kendi `Review trigger`'ını (ya da varsayılan olarak bir sonraki backlog faz geçişini) kontrol eder, takvime ya da birinin hatırlamasına güvenmek yerine.
- **Metodolojiyi sürdürür.** `skill/DECISIONS.md` — skill kendi disiplinini kendi evrimine uyguluyor.

Bu beş görevin hiçbiri tek tek yeni değil — aşağıdaki orchestrator/supervisor karşılaştırmasına bakın. Daha az yaygın olan, bu rolü bu kadar somut, düz dosyalarla, belirli bir orkestrasyon framework'ü gerektirmeden tanımlamış olmak.

Ayrımı net koymakta fayda var: **Project Historian kavramın kendisi — rol tanımı. Precedent ise onun bir implementasyonu** — tıpkı "staging area" ya da "control plane"in, farklı araçların farklı biçimlerde gerçekleştirebileceği bir örüntüye isim vermesi gibi. Aynı rolü farklı bir stack'te, farklı dosya isimleriyle kuran biri de Project Historian rolünü implemente ediyor olurdu — bunu yapmak için bu repoya ihtiyacı olmazdı.

## Sözlük

Bu repo boyunca tekrar eden birkaç terim var, her yerde aynı anlama geliyor:

- **Project Historian** — rol: bir projenin karar geçmişini koruyan, hatırlayan, yöneten, gözden geçiren ve sürdüren, işi yürüten ajan(lar)dan ayrı bir ajan.
- **Command Center** — iş akışının planlama tarafı: kararların alındığı ve kayıt altına alındığı yer, kodun yazıldığı yerin karşıtı.
- **Decision Draft** — bir sapmayı ya da o anki kararı yakalayan, henüz triaj edilmemiş, formatsız ham bir not.
- **Decision Routing** — asıl çözülen problem: bir kararın, gerçekten ihtiyaç duyulduğu yere ve ana güvenilir şekilde ulaşmasını sağlamak. Hafıza değil (geçmişi geri çekmek) — routing (geleceğe teslim etmek).
- **Decision Lifecycle** — bir kararın zaman içinde geçtiği durumlar (proposed, accepted, implemented, validated, ya da rejected/deprecated/superseded), düz bir "alındı/alınmadı" yerine.

Bunlar "staging area" ya da "control plane" gibi bu reponun dışında yerleşik terimler değil — sadece her yerde tutarlı kalsınlar diye burada bir kere tanımlandı, benimsenmiş bir sözlük olarak iddia edilmiyor.

## Nasıl çalışıyor

Her proje şu doküman setiyle başlar:

```
<proje>/
  AGENTS.md                ← proje briefi, <200 satır
  docs/
    PRD.md                  ← ne
    TECH_SPEC.md             ← nasıl
    DECISIONS.md             ← neden (karar günlüğü)
    DECISIONS_DRAFT.md       ← inbox: işlenmemiş karar notları
    BACKLOG.md               ← ne zaman / hangi sırayla
  briefs/
    DESIGN_BRIEF.md          ← tasarım gerekiyorsa
```

Bu yapının sabit bir kök yolu yok. `project-init`, çalıştırıldığı anda ajana hangi klasör bağlıysa oraya yazar; kategori yapısını da varsaymaz, senin tanımlamanı ister (mobile/saas/her ne ise — bu sana kalmış, koda gömülü değil). Pratikte bu şu demek: çalıştırmadan önce gerçek proje workspace'ini bağla, bu reponun kendisini değil. Bu repo bağlıyken çalıştırırsan, proje klasörlerini Precedent'ın içine açmaya çalışır — bu repo temiz bir referans/şablon olarak kalmalı, üretilen projeler birikmemeli.

Akış tek yönlü bir kapı üzerinden işler: planlama ajanı, karar `docs/`'a yazılmadan kod ajanına çalıştırılabilir bir prompt vermez. Bu, bir kararın kayda hiç uğramadan koda sızmasını yapısal olarak engeller.

Bu tek yönlü akış tek başına yeterli değil — uygulamadan planlamaya geri dönen bir kanal da gerekiyor, yoksa dokümanlar sessizce gerçeklikten kopar. Bunun için düşük sürtünmeli bir inbox kullanılıyor: kod ajanı implementasyon sırasında brief'ten saptığında, `DECISIONS_DRAFT.md`'ye ham bir satır not düşer. Bir sonraki oturumun başında planlama ajanı her notu üç sonuçtan birine triaj eder — Accepted (tam bir ADR'ye yükseltilir), Rejected (kaybolmak yerine tek satırlık bir gerekçeyle kayda geçer), ya da Ignored (gerçek gürültü, iz bırakmadan temizlenir).

Geçmiş kararların bayatlaması da farklı ele alınıyor: her ADR kendi `Review trigger`'ını taşıyabilir — bir metrik, çıkan bir versiyon, o karar için gerçekten önemli olan her ne ise — genel bir takvim hatırlatması yerine. Bir karar bunu belirtmezse, varsayılan olarak `BACKLOG.md`'deki bir sonraki faz geçişine (Faz 0 → 1 → 2) bağlanır. Her iki durumda da, ilerlemeden önce o ana kadar alınan kararların dayandığı varsayımlar hızlıca kontrol edilir.

## Temel ilkeler

- **Önce netleştir ve dokümante et, sonra uygula.** Kod yazmadan önce ne, neden ve en büyük risk yazıya dökülür.
- **Her önemli karar gerekçesiyle kaydedilir.** Karar = ne + neden + sonuçlar.
- **Doğrulanmamış varsayımlar gerçekmiş gibi sunulmaz.**
- **Blokajlar mimariden önce çözülür.** Projenin kalbindeki belirsizlik, üzerine inşa edilmeden önce test edilir.
- **Kararlar tek yönlü akmaz.** Planlamadan uygulamaya giden kanal kadar, geri dönen kanal da önemli.
- **Bağlam ekonomisi.** Ajanın context'ine sadece çekirdek dokümanlar (PRD + TECH_SPEC + DECISIONS) girer; ham loglar ve deney çıktısı dışarıda kalır.

## Bilinen pratiklerle ilişki

Dürüst olmak için, her parçanın gerçekte nereden geldiği burada.

| Bu metodolojide | Bilinen karşılığı | Not |
|---|---|---|
| **Project Historian** (rol) | Orchestrator / supervisor agent pattern | Multi-agent framework'lerde zaten yerleşik (LangGraph'ın supervisor deseni, CrewAI'nin manager agent'ı). Buradaki fark, hiçbir orkestrasyon framework'ü gerektirmeden — sadece iki sohbet tabanlı ajan ve paylaşılan bir klasörle — beş görevi somut, düz dosyalarla tarif etmiş olmamız. |
| `DECISIONS.md` | Architecture Decision Record (ADR) | Michael Nygard, 2011. Format (Bağlam→Karar→Sonuçlar→Statü) Nygard'ın orijinal şablonunun neredeyse birebir çevirisi. |
| `AGENTS.md` | AGENTS.md | Ağustos 2025'te OpenAI, Google, Cursor ve Factory'nin kurduğu, Aralık 2025'te Linux Foundation'a devredilen açık standart. 60.000+ projede kullanılıyor. |
| PRD → TECH_SPEC → BACKLOG sırası | Spec-Driven Development / GitHub Spec-Kit | GitHub'ın 2025–2026 araç seti aynı sırayı öneriyor: önce spec, sonra plan, sonra görevler. |
| Bağlam ekonomisi | Context engineering | Anthropic'in Eylül 2025 mühendislik yazısı: "en küçük, en yüksek sinyalli token kümesini bul." |
| `DECISIONS_DRAFT.md` (inbox) | Structured note-taking / agentic memory | Aynı Anthropic yazısında tarif edilen desen: ajan context dışında bir NOTES.md tutar, sonra geri okur. |

## Bilinen riskler / açık sorular

- **ADR'ler yaygın olarak terk ediliyor.** Literatür net: takımlar coşkuyla başlayıp birkaç ay içinde sessizce bırakıyor. Sebepler arasında bakım yükünün en az bağlama sahip kişiye düşmesi, önemsiz kararların önemli olanları gölgeleyip güveni eritmesi, ve kaydın günlük iş akışının dışında yaşaması var. Tek-yönlü-kapı tasarımı kaydın iş akışı dışında yaşaması sorununu hedefliyor; Importance alanı önemsiz kararların önemlileri gölgelemesi sorununu hedefliyor. Bakım yükünün en az bağlama sahip kişiye düşmesi sorunu hâlâ açık.
- **Inbox'un "poisoning" riski.** Bir seferlik, önemsiz bir sapmanın kalıcı bir karar kaydına yükselmesi, ajana yanlış bir dersi kalıcı olarak öğretebilir. Küratörlük (planlama ajanının neyin kayda gireceğine karar vermesi) bu riski azaltıyor ama ortadan kaldırmıyor.
- **Ölçek test edilmedi.** Bu metodoloji az sayıda, seyrek kararlı proje için tasarlandı. Ekip/hacim ölçeğinde nerede kırılacağı açık bir soru.

## Kullanım

`skill/SKILL.md`, doğrudan bir Claude/Cowork skill'i olarak `project-init` adıyla kurulabilir (Project Historian rolünün bir implementasyonu olan repo, Precedent olarak adlandırıldı; skill'in kendisi ise `git init` veya `npm init` gibi ne yaptığına göre adlandırıldı). `skill/DECISIONS.md`, skill'in kendi değişiklik geçmişini ADR formatında tutar.

## Kaynaklar

- Michael Nygard, "Documenting Architecture Decisions" (2011) — https://adr.github.io/
- AGENTS.md — https://agents.md/
- GitHub Spec-Kit — https://github.com/github/spec-kit
- Anthropic, "Effective context engineering for AI agents" (2025) — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic, "Claude Code best practices" — https://www.anthropic.com/engineering/claude-code-best-practices
- "The reason most ADRs get written and never read" — https://www.javacodegeeks.com/2026/05/the-reason-most-architecture-decision-records-get-written-and-never-read-is-architectural-not-cultural.html
