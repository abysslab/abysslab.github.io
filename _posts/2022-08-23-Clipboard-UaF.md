---
layout: post
author: "nevul37"
title: Real case of UaF in Clipboard 
---
[reference](https://bugs.chromium.org/p/chromium/issues/detail?id=1101509)  


Introduction
---
Issue 1101509를 짧게 소개하려고 한다.
이 취약점은 RawClipboardHostImpl의 lifetime 때문에 일어나는 취약점이다.
  
Root Causes
---
ClipboardHostImpl은 다음과 같이 이루어져 있다.
```c++
ClipboardHostImpl::ClipboardHostImpl(
    RenderFrameHost* render_frame_host,
    mojo::PendingReceiver<blink::mojom::ClipboardHost> receiver)
    : receiver_(this, std::move(receiver)),
      clipboard_(ui::Clipboard::GetForCurrentThread()),
      clipboard_writer_(
          new ui::ScopedClipboardWriter(ui::ClipboardBuffer::kCopyPaste)) {
  // |render_frame_host| may be null in unit tests.
  if (render_frame_host) {
    render_frame_routing_id_ = render_frame_host->GetRoutingID();
    render_frame_pid_ = render_frame_host->GetProcess()->GetID();
  } else {
    render_frame_routing_id_ = MSG_ROUTING_NONE;
    render_frame_pid_ = ChildProcessHost::kInvalidUniqueID;
  }
}
void ClipboardHostImpl::Create(
    RenderFrameHost* render_frame_host,
    mojo::PendingReceiver<blink::mojom::ClipboardHost> receiver) {
  // Clipboard implementations do interesting things, like run nested message
  // loops. Use manual memory management instead of SelfOwnedReceiver<T> which
  // synchronously destroys on failure and can result in some unfortunate
  // use-after-frees after the nested message loops exit.
  auto* host = new ClipboardHostImpl(
      static_cast<RenderFrameHostImpl*>(render_frame_host),
      std::move(receiver));
  host->receiver_.set_disconnect_handler(base::BindOnce(
      [](ClipboardHostImpl* host) {
        base::SequencedTaskRunnerHandle::Get()->DeleteSoon(FROM_HERE, host);
      },
      host));
}

```
ClipboardHostImpl이 RenderFrameHost를 `raw pointer`로 갖고 있다. Create 함수에서 ClipboardHostImpl 객체를 생성할 때 `static_cast<RenderFrameHostImpl*>(render_frame_host)`을 인자로 넘겨주는 것을 확인할 수 있다. 해당 객체가 해제될 경우 `render_frame_host` 또한 같이 메모리에서 해제되어야 하지만 lifetime에 대한 어떠한 처리도 되어있지 않아 UaF가 발생하게 된다. 보통은 WebContentsObserver을 상속받게 하여 `render_frame_host`와 `Mojo interface`의 lifetime을 동일하게 맞춰주는 방법으로 패치를 진행한다.
```
PoC
---
