---
layout: post
title: "Flutter #30: BLoC — Event, State & BlocBuilder"
date: 2026-09-08 07:00:00 +0700
tags: [flutter, dart, tutorial, bloc, event, state, blocbuilder, state-management, clean-architecture]
description: "Upgrade dari Cubit ke BLoC penuh! Pelajari Event, State class terpisah, BlocBuilder, dan bagaimana BLoC membuat flow data jadi predictable dan testable."
---

# Flutter #30: BLoC — Event, State & BlocBuilder

Halo! Di artikel #29 kita sudah kenal **Cubit** — versi ringkas dari BLoC yang simpel dan powerful. Tapi begitu logic-nya makin kompleks, Cubit punya kelemahan: **tidak ada batasan tindakan**. Siapa aja bisa panggil `increment()` kapan aja — gak ada trace, gak ada history.

Masuklah **BLoC penuh** dengan **Event** dan **State** yang terpisah. Pattern ini bikin flow data jadi: **Event masuk → BLoC proses → State keluar**. Terstruktur, predictable, dan gampang di-debug.

Let's go!

---

## 1. Cubit vs BLoC: Bedanya Apa?

| Aspek | Cubit | BLoC |
|-------|-------|------|
| Input | Fungsi biasa (`increment()`) | Event (`IncrementPressed`) |
| State change | `emit(state.copyWith(...))` | `emit(Loading → Success)` |
| Traceability | Gak ada history | Event bisa di-log, replay, test |
| Complexity | Cocok untuk simple logic | Cocok untuk complex workflow |

**Rule of thumb:** Kalau logic-nya cuma "kalau X, emit Y" → pakai Cubit. Kalau butuh "kalau event A, proses async, bisa gagal, emit loading → success/error" → pakai BLoC.

---

## 2. Arsitektur Event-Driven

Ini flow data di BLoC:

```
UI ─── dispatch(IncrementPressed()) ───→ BLoC ─── emit(CounterState(count: 1)) ───→ UI
      ╰── Event ──────────────────────╯     ╰── Proses ──────────────────────╯
```

1. **Event** — aksi dari user (tombol ditekan, form disubmit, API dipanggil)
2. **BLoC** — terima event, proses logic, emit state baru
3. **State** — representasi condition UI saat ini
4. **UI** — render berdasarkan state

Keuntungan? Setiap perubahan state bisa di-trace ke event tertentu. Debugging jadi jauh lebih gampang!

---

## 3. Setup Package

Masih sama seperti Cubit — kita pakai `flutter_bloc`:

```yaml
# pubspec.yaml
dependencies:
  flutter_bloc: ^8.1.0
  equatable: ^2.0.5
  freezed_annotation: ^2.4.1  # optional, tapi membantu

dev_dependencies:
  build_runner: ^2.4.0
  freezed: ^2.4.0              # optional
```

---

## 4. Contoh Pertama: Counter BLoC

Bikin 3 file terpisah — Event, State, dan BLoC:

### Event (`counter_event.dart`)

```dart
import 'package:equatable/equatable.dart';

// Semua event turunan dari CounterEvent
abstract class CounterEvent extends Equatable {
  const CounterEvent();

  @override
  List<Object?> get props => [];
}

// Event ketika user pencet tombol tambah
class CounterIncremented extends CounterEvent {
  const CounterIncremented();
}

// Event ketika user pencet tombol kurang
class CounterDecremented extends CounterEvent {
  const CounterDecremented();
}

// Event ketika user pencet reset
class CounterReset extends CounterEvent {
  const CounterReset();
}
```

### State (`counter_state.dart`)

```dart
import 'package:equatable/equatable.dart';

class CounterState extends Equatable {
  final int count;
  final String status; // 'initial', 'active', 'reset'

  const CounterState({
    this.count = 0,
    this.status = 'initial',
  });

  CounterState copyWith({int? count, String? status}) {
    return CounterState(
      count: count ?? this.count,
      status: status ?? this.status,
    );
  }

  @override
  List<Object?> get props => [count, status];
}
```

### BLoC (`counter_bloc.dart`)

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_event.dart';
import 'counter_state.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState()) {
    // Register handler untuk setiap event
    on<CounterIncremented>(_onIncrement);
    on<CounterDecremented>(_onDecrement);
    on<CounterReset>(_onReset);
  }

  // Handler untuk CounterIncremented
  void _onIncrement(
    CounterIncremented event,
    Emitter<CounterState> emit,
  ) {
    emit(state.copyWith(
      count: state.count + 1,
      status: 'active',
    ));
  }

  // Handler untuk CounterDecremented
  void _onDecrement(
    CounterDecremented event,
    Emitter<CounterState> emit,
  ) {
    final newCount = state.count > 0 ? state.count - 1 : 0;
    emit(state.copyWith(
      count: newCount,
      status: newCount == 0 ? 'initial' : 'active',
    ));
  }

  // Handler untuk CounterReset
  void _onReset(
    CounterReset event,
    Emitter<CounterState> emit,
  ) {
    emit(const CounterState(count: 0, status: 'reset'));
  }
}
```

**Perhatikan polanya:**

1. **`on<EventType>(_handler)`** — daftarkan handler untuk setiap event. Beda sama Cubit yang pakai fungsi biasa.
2. **`Emitter<CounterState>`** — param `emit` di BLoC lebih aman, hanya bisa diakses di dalam handler.
3. **`_onIncrement`** — naming convention: `_on` + nama event.

---

## 5. BlocProvider — Tanam BLoC ke Tree

Sama seperti Cubit, kita pakai `BlocProvider`:

```dart
void main() {
  runApp(
    BlocProvider(
      create: (_) => CounterBloc(),
      child: const MyApp(),
    ),
  );
}
```

Tapi bedanya, sekarang UI mengirim **Event**, bukan memanggil fungsi langsung:

```dart
// Cubit (sebelumnya)
context.read<CounterCubit>().increment();

// BLoC (sekarang)
context.read<CounterBloc>().add(const CounterIncremented());
```

Yang dikirim ke BLoC adalah **event object**, bukan method call. Inilah yang bikin BLoC lebih traceable!

---

## 6. BlocBuilder — Render UI dari State

`BlocBuilder` adalah widget utama untuk render UI berdasarkan state:

```dart
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter BLoC')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // BlocBuilder rebuild hanya saat state berubah
            BlocBuilder<CounterBloc, CounterState>(
              buildWhen: (previous, current) {
                // Cuma rebuild kalau count berubah
                return previous.count != current.count;
              },
              builder: (context, state) {
                return Column(
                  children: [
                    Text(
                      '${state.count}',
                      style: const TextStyle(
                        fontSize: 72,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'Status: ${state.status}',
                      style: TextStyle(
                        fontSize: 16,
                        color: state.status == 'reset'
                            ? Colors.orange
                            : Colors.grey,
                      ),
                    ),
                  ],
                );
              },
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(
                      const CounterDecremented(),
                    );
                  },
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(
                      const CounterIncremented(),
                    );
                  },
                  child: const Icon(Icons.add),
                ),
                const SizedBox(width: 16),
                OutlinedButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(
                      const CounterReset(),
                    );
                  },
                  child: const Text('Reset'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

**Yang penting:**
- **`BlocBuilder`** cuma rebuild bagian widget yang berubah
- **`buildWhen`** memberikan kontrol eksplisit kapan rebuild terjadi
- **`context.read<CounterBloc>().add(...)`** — kirim event ke BLoC

---

## 7. BlocListener — Side Effects

Mau tampilkan SnackBar ketika state tertentu? Pakai `BlocListener`:

```dart
BlocListener<CounterBloc, CounterState>(
  listenWhen: (previous, current) {
    // Cuma listen kalau status berubah ke 'reset'
    return previous.status != current.status &&
           current.status == 'reset';
  },
  listener: (context, state) {
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text('Counter sudah di-reset! 🔃'),
        backgroundColor: Colors.orange,
      ),
    );
  },
  child: const CounterBody(), // child tetap rebuild normal
),
```

`BlocListener` **tidak rebuild** child-nya — dia cuma eksekusi side-effect. Berbeda dengan `BlocBuilder`.

---

## 8. Contoh Nyata: Async BLoC dengan Loading/Error

Ini di mana BLoC bersinar — handle **async operation** dengan state yang terstruktur:

```dart
// === State: punya 3 kondisi ===
abstract class UserState extends Equatable {
  const UserState();
  @override
  List<Object?> get props => [];
}

class UserInitial extends UserState {}
class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final String name;
  final String email;
  const UserLoaded({required this.name, required this.email});
  @override
  List<Object?> get props => [name, email];
}

class UserError extends UserState {
  final String message;
  const UserError({required this.message});
  @override
  List<Object?> get props => [message];
}

// === Event ===
class UserFetched extends UserEvent {
  final int userId;
  const UserFetched({required this.userId});
  @override
  List<Object?> get props => [userId];
}

// === BLoC ===
class UserBloc extends Bloc<UserEvent, UserState> {
  final ApiRepository repository;

  UserBloc({required this.repository}) : super(UserInitial()) {
    on<UserFetched>(_onUserFetched);
  }

  Future<void> _onUserFetched(
    UserFetched event,
    Emitter<UserState> emit,
  ) async {
    // 1. Emit Loading
    emit(UserLoading());

    try {
      // 2. Fetch data dari API
      final user = await repository.fetchUser(event.userId);

      // 3. Emit Success
      emit(UserLoaded(name: user.name, email: user.email));
    } catch (e) {
      // 4. Emit Error
      emit(UserError(message: 'Gagal load user: $e'));
    }
  }
}
```

UI-nya jadi super clean:

```dart
BlocBuilder<UserBloc, UserState>(
  builder: (context, state) {
    if (state is UserLoading) {
      return const CircularProgressIndicator();
    }
    if (state is UserError) {
      return Text('Error: ${state.message}');
    }
    if (state is UserLoaded) {
      return Text('Halo, ${state.name}!');
    }
    return const Text('Tekan tombol untuk load user');
  },
);
```

**Inilah kekuatan BLoC:** UI tinggal handle 4 kondisi — `Initial`, `Loading`, `Loaded`, `Error`. Gak ada logic yang bocor ke UI!

---

## 9. Event Transformer — Debounce & Throttle

BLoC punya fitur keren: **transform event stream**. Contoh: debounce untuk search input:

```dart
class SearchBloc extends Bloc<SearchEvent, SearchState> {
  SearchBloc() : super(SearchInitial()) {
    // Debounce: tunggu user berhenti ketik 300ms baru proses
    on<SearchQueryChanged>(
      _onSearchChanged,
      transformer: debounce(Duration(milliseconds: 300)),
    );
  }

  Future<void> _onSearchChanged(
    SearchQueryChanged event,
    Emitter<SearchState> emit,
  ) async {
    if (event.query.isEmpty) {
      emit(SearchInitial());
      return;
    }

    emit(SearchLoading());
    try {
      final results = await apiService.search(event.query);
      emit(SearchLoaded(results: results));
    } catch (e) {
      emit(SearchError(message: 'Search failed'));
    }
  }
}

// Helper function untuk debounce
EventTransformer<E> debounce<E>(Duration duration) {
  return (events, mapper) {
    return events.debounceTime(duration).asyncExpand(mapper);
  };
}
```

Tanpa debounce, setiap ketikan trigger API call. Dengan debounce, cuma trigger setelah user berhenti 300ms. Hemat resource!

---

## 10. Perbandingan Cubit vs BLoC — Kapan Pakai Apa?

| Skenario | Pilih | Alasan |
|----------|-------|--------|
| Counter, Toggle, Simple state | Cubit | Lebih sedikit boilerplate |
| Form submit + validation | BLoC | Perlu Loading/Success/Error state |
| API call dengan cache | BLoC | Event transformer, complex flow |
| Theme switcher | Cubit | Simpel, cuma emit state |
| Search dengan debounce | BLoC | Transformer built-in |
| Auth flow (login, register, logout) | BLoC | Multiple async steps, error handling |

**Prinsipnya:** Mulai dari Cubit. Kalau mentok, naik ke BLoC. Jangan over-engineer dari awal!

---

## Kesimpulan

Hari ini kita belajar:

1. **Event** — aksi yang dikirim dari UI ke BLoC
2. **State** — kondisi yang di-emit BLoC ke UI
3. **BLoC** — register handler per event, proses logic, emit state
4. **`BlocBuilder`** — render UI dari state
5. **`BlocListener`** — eksekusi side-effect tanpa rebuild
6. **Async pattern** — `Loading → Loaded/Error` yang clean
7. **Event Transformer** — debounce & throttle untuk optimasi

Di artikel #31 kita lanjut ke **GetX** — state management all-in-one yang pakai syntax super ringkas. Beda banget sama BLoC!

---

**Coba sendiri!** Buat BLoC untuk fetch data dari `jsonplaceholder.typicode.com/users`, tampilkan di ListView dengan state Loading/Success/Error. Share hasilnya ke sosial media dan tag **@ahsai001** 🚀
