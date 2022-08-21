---
layout: post
author: "nevul37"
title: Real case of UaF in Clipboard 
---
[reference](https://bugs.chromium.org/p/chromium/issues/detail?id=1101509)  


Introduction
---
beta 버전에서 난 취약점이라 CVE는 아니지만 UaF가 일어나기 좋은 케이스를 보여준 Issue 1101509를 짧게 소개하려고 한다.
이 취약점은 RawClipboardHostImpl의 lifetime 때문에 일어나는 취약점이며 간단하게 트리거가 가능하다. (poc는 reference를 참고.)
  
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
생성자 안에 RenderFrameHost의 raw pointer가 선언되어 있고 객체를 생성할 때 또한 render_frame_host를 이용한다. 해당 코드에는 raw pointer와 object 사이에 lifetime issue가 존재한다. render_frame_host가 free 되면 object도 free 되어야 하지만, 아무런 조치를 취해주지 않아 RawClipboardHostImpl 객체는 dangling pointer를 갖고 있게 되며 이 dangling pointer를 이용한 함수를 호출 시, UaF가 발생하게 된다.  
이러한 경우를 방지하기 위해 보통은 Mojo interface가 WebcontentsObserver를 상속하여 RenderFrameDeleted를 재정의하거나, FrameServicebase를 사용해서 해결하곤 한다.

